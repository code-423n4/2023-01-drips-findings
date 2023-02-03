
| S No. | Issue | Instances | Gas Savings (from provided tests) |
|-----|-----|-----|-----|
| [G-01] | x += y costs more gas than x = x + y for state variables (same applies for x -= y)| 21 | 2373
| [G-02] | Use msg.sender instead of openzeppelin's _msgsender()_ when meta-transactions capabailities aren't used | 6 | 12
| [G-03] | Internal functions only called once can be inlined to save gas | 15 | 600
| [G-04] | Increments can be unchecked | 16 | 640
| [G-05] | Using both named returns and a return statement isn't necessary | 48 | 144

-------------

## G-01 x += y costs more gas than x = x + y for state variables (same applies for x -= y)

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

_There are 21 instances of this issue_

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol

```
File: src/Drips.sol

253: amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
284: toCycle -= receivableCycles;
288: amtPerCycle += state.amtDeltas[cycle].thisCycle;
289: receivedAmt += uint128(amtPerCycle);
290: amtPerCycle += state.amtDeltas[cycle].nextCycle;
429: amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);
495: amt += _drippedAmt(receiver.config.amtPerSec(), start, end);
572: balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
755: spent += _drippedAmt(amtPerSec, start, end);
1053: amtDelta.thisCycle += int128(fullCycle - nextCycle);
1054: amtDelta.nextCycle += int128(nextCycle);
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol

```
File: src/DripsHub.sol

632: totalBalances[erc20] += amt;
636: _dripsHubStorage().totalBalances[erc20] -= amt;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol

```
File: src/ImmutableSplitsDriver.sol

62: weightSum += receivers[i].weight;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol

```
File: src/Splits.sol

99:  _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
128: splitsWeight += currReceivers[i].weight;
159: splitsWeight += currReceivers[i].weight;
162: splitAmt += currSplitAmt;
167: collectableAmt -= splitAmt;
168: balance.collectable += collectableAmt;
235: totalWeight += weight;
```

----------

## G-02 Use msg.sender instead of openzeppelin's _msgsender()_ when meta-transactions capabailities aren't used

`msg.sender` costs **2 gas (CALLER opcode)**. `_msgSender()` represents the following:

```
function _msgSender() internal view virtual returns (address payable) {  return msg.sender;}
```

When no meta-transactions capabilities are used: `msg.sender` is enough.

See [https://docs.openzeppelin.com/contracts/2.x/gsn](https://docs.openzeppelin.com/contracts/2.x/gsn) for more information about GSN capabilities.

Consider replacing `_msgSender()` with `msg.sender` here

_There are 6 instances of this issue_

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol

```
File: src/AddressDriver.sol

47: return calcUserId(_msgSender());
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

```
File: src/Caller.sol

107: address sender = _msgSender();
115: address sender = _msgSender();
149: require(isAuthorized(sender, _msgSender()), "Not authorized");
195: address sender = _msgSender();
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol

```
File: src/NFTDriver.sol

42:  _isApprovedOrOwner(_msgSender(), tokenId),
```

----

## G-03 Internal functions only called once can be inlined to save gas

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

_There are 15 instances of this issue_

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol

```
File: src/Drips.sol

83: function start(DripsConfig config) internal pure returns (uint32) {
88: function duration(DripsConfig config) internal pure returns (uint32) {
94: function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {
233: function _receiveDrips(uint256 userId, uint256 assetId, uint32 maxCycles)
300: function _receivableDripsCycles(uint256 userId, uint256 assetId)
342: function _squeezeDrips(
508: function _dripsState(uint256 userId, uint256 assetId)
610: function _setDrips(
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol

```
File: src/Splits.sol

106: function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
117: function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
143: function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)
176: function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
185: function _collect(uint256 userId, uint256 assetId) internal returns (uint128 amt) {
199: function _give(uint256 userId, uint256 receiver, uint256 assetId, uint128 amt) internal {
212: function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {
```

-----

## G-04 Increments can be unchecked

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used. It saves **30-40 gas** per loop.

_There are 16 instances of this issue_

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

```
File: src/Caller.sol

196: for (uint256 i = 0; i < calls.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol

```
File: src/Drips.sol

247: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
287: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
358: for (uint256 i = 0; i < squeezedNum; i++) {
422: for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
450: for (uint256 i = 0; i < dripsHistory.length; i++) {
490: for (; idx < receivers.length; idx++) {
563: for (uint256 i = 0; i < receivers.length; i++) {
664: for (uint256 i = 0; i < newReceivers.length; i++) {
745: for (uint256 i = 0; i < configsLen; i++) {
777: for (uint256 i = 0; i < receivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol

```
File: src/DripsHub.sol

613: for (uint256 i = 0; i < userMetadata.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol

```
File: src/ImmutableSplitsDriver.sol

61: for (uint256 i = 0; i < receivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol

```
File: src/Splits.sol

127: for (uint256 i = 0; i < currReceivers.length; i++) {
158: for (uint256 i = 0; i < currReceivers.length; i++) {
231: for (uint256 i = 0; i < receivers.length; i++) {
```

---

## G-05 Using both named returns and a return statement isn't necessary 

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.

Each MLOAD and MSTORE costs **3 additional gas**

_There are 48 instances of this issue_

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol

```
File: src/AddressDriver.sol

40: function calcUserId(address userAddr) public view returns (uint256 userId) {
46: function callerUserId() internal view returns (uint256 userId) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

```
File: src/Caller.sol

124: function isAuthorized(address sender, address user) public view returns (bool authorized) {
132: function allAuthorized(address sender) public view returns (address[] memory authorized) {
144-147: function callAs(address sender, address to, bytes memory data)
        public
        payable
        returns (bytes memory returnData)
164-171: function callSigned(
        address sender,
        address to,
        bytes memory data,
        uint256 deadline,
        bytes32 r,
        bytes32 sv
    ) public payable returns (bytes memory returnData) {
202-204: function _call(address sender, address to, bytes memory data, uint256 value)
        internal
        returns (bytes memory returnData)
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol

```
File: src/Drips.sol

60-63: function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)
        internal
        pure
        returns (DripsConfig)
300-303: function _receivableDripsCycles(uint256 userId, uint256 assetId)
        internal
        view
        returns (uint32 cycles)
470-475: function _squeezedAmt(
        uint256 userId,
        DripsHistory memory dripsHistory,
        uint32 squeezeStartCap,
        uint32 squeezeEndCap
    ) private view returns (uint128 squeezedAmt) {
508-511: function _dripsState(uint256 userId, uint256 assetId)
        internal
        view
        returns (
533-538: function _balanceAt(
        uint256 userId,
        uint256 assetId,
        DripsReceiver[] memory receivers,
        uint32 timestamp
    ) internal view returns (uint128 balance) {
737-742: function _isBalanceEnough(
        uint256 balance,
        uint256[] memory configs,
        uint256 configsLen,
        uint256 maxEnd
    ) private view returns (bool isEnough) {
792-795: function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver)
        private
        view
        returns (uint256 newConfigsLen)
815-818: function _getConfig(uint256[] memory configs, uint256 idx)
        private
        pure
        returns (uint256 amtPerSec, uint256 start, uint256 end)
834-837: function _hashDrips(DripsReceiver[] memory receivers)
        internal
        pure
        returns (bytes32 dripsHash)
852-857: function _hashDripsHistory(
        bytes32 oldDripsHistoryHash,
        bytes32 dripsHash,
        uint32 updateTime,
        uint32 maxEnd
    ) internal pure returns (bytes32 dripsHistoryHash) {
970-973: function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)
        private
        view
        returns (uint32 start, uint32 end)
985-991: function _dripsRange(
        DripsReceiver memory receiver,
        uint32 updateTime,
        uint32 maxEnd,
        uint32 startCap,
        uint32 endCap
    ) private pure returns (uint32 start, uint32 end_) {
1120: function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
1128: function _currTimestamp() private view returns (uint32 timestamp) {
1134: function _currCycleStart() private view returns (uint32 timestamp) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol

```
File: src/DripsHub.sol

145: function driverAddress(uint32 driverId) public view returns (address driverAddr) {
160: function nextDriverId() public view returns (uint32 driverId) {
169: function cycleSecs() public view returns (uint32 cycleSecs_) {
181: function totalBalance(IERC20 erc20) public view returns (uint256 balance) {
196-199: function receivableDripsCycles(uint256 userId, IERC20 erc20)
        public
        view
        returns (uint32 cycles)
317: function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
328-331: function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
        public
        view
        returns (uint128 collectableAmt, uint128 splitAmt)
355-358: function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)
        public
        whenNotPaused
        returns (uint128 collectableAmt, uint128 splitAmt)
372: function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
432-435: function dripsState(uint256 userId, IERC20 erc20)
        public
        view
        returns (
460-465: function balanceAt(
        uint256 userId,
        IERC20 erc20,
        DripsReceiver[] memory receivers,
        uint32 timestamp
    ) public view returns (uint128 balance) {
546: function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {
557-562: function hashDripsHistory(
        bytes32 oldDripsHistoryHash,
        bytes32 dripsHash,
        uint32 updateTime,
        uint32 maxEnd
    ) public pure returns (bytes32 dripsHistoryHash) {
587: function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {
595-598: function hashSplits(SplitsReceiver[] memory receivers)
        public
        pure
        returns (bytes32 receiversHash)
642: function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol

```
File: src/ImmutableSplitsDriver.sol

36: function nextUserId() public view returns (uint256 userId) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol

```
File: src/Managed.sol

105: function isPauser(address pauser) public view returns (bool isAddrPauser) {
112: function allPausers() public view returns (address[] memory pausersList) {
117: function paused() public view returns (bool isPaused) {
136: function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol

```
File: src/NFTDriver.sol

50: function nextTokenId() public view returns (uint256 tokenId) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol

```
File: src/Splits.sol

106: function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
176: function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
262: function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {
270-273: function _hashSplits(SplitsReceiver[] memory receivers)
        internal
        pure
        returns (bytes32 receiversHash)
```

------

