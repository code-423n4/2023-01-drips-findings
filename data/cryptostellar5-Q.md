
### 01 UPGRADEABLE CONTRACT IS MISSING A `__GAP[50]` STORAGE VARIABLE TO ALLOW FOR NEW STORAGE VARIABLES IN LATER VERSIONS

*Number of Instances Identified: 1*

See [this]([https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps)) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.


https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol

```
18: abstract contract Managed is UUPSUpgradeable 
```


### 02 ADDING A RETURN STATEMENT WHEN THE FUNCTION DEFINES A NAMED RETURN VARIABLE, IS REDUNDANT

*Number of Instances Identified: 48*

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol

```
40: function calcUserId(address userAddr) public view returns (uint256 userId) {
46: function callerUserId() internal view returns (uint256 userId) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

```
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
36: function nextUserId() public view returns (uint256 userId) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol

```
105: function isPauser(address pauser) public view returns (bool isAddrPauser) {
112: function allPausers() public view returns (address[] memory pausersList) {
117: function paused() public view returns (bool isPaused) {
136: function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol

```
50: function nextTokenId() public view returns (uint256 tokenId) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol

```
106: function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
176: function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
262: function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {
270-273: function _hashSplits(SplitsReceiver[] memory receivers)
        internal
        pure
        returns (bytes32 receiversHash)
```


### 03 MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS

*Number of Instances Identified: 7*

TThe functions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol

```
122: function setDrips(
158: function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol

```
510: function setDrips(
576: function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol

```
186: function setDrips(
220: function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)
272: function setApprovalForAll(address operator, bool approved) public override whenNotPaused {
```


### 04 FLOATING PRAGMA VERSION SHOULD NOT BE USED

*Number of Instances Identified: 8*

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

### Proof of Concept

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol

```
2: pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

```
2: pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol

```
2: pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol

```
2: pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol

```
2: pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol

```
2: pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol

```
2: pragma solidity ^0.8.17;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol

```
2: pragma solidity ^0.8.17;
```


### 05 INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

*Number of Instances Identified: 12*

### Description

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

If Return parameters are declared, you must prefix them with `/// @return`.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the `@return` tag, they do incomplete analysis.

### Recommendation

Include return parameters in NatSpec comments

Recommendation Code Style:

```
  /// @return Return value description
```

POC

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

```
202-204: function _call(address sender, address to, bytes memory data, uint256 value)
        internal
        returns (bytes memory returnData)
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol

```
63: returns (DripsConfig)
73-74: function dripId(DripsConfig config) internal pure returns (uint32) {
        return uint32(DripsConfig.unwrap(config) >> 224);
78-79: function amtPerSec(DripsConfig config) internal pure returns (uint160) {
        return uint160(DripsConfig.unwrap(config) >> 64);
83-84: function start(DripsConfig config) internal pure returns (uint32) { 
        return uint32(DripsConfig.unwrap(config) >> 32);
88-89: function duration(DripsConfig config) internal pure returns (uint32) { 
        return uint32(DripsConfig.unwrap(config));
94-95: function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) { 
        return DripsConfig.unwrap(config) < DripsConfig.unwrap(otherConfig);
1061-1064: function _isOrdered(DripsReceiver memory prev, DripsReceiver memory next)
        private
        pure
        returns (bool)
1093-1096: function _drippedAmt(uint256 amtPerSec, uint256 start, uint256 end)
        private
        view
        returns (uint256 amt)
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol

```
78-79: function admin() public view returns (address) {
        return _getAdmin();
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol

```
294-295: function _msgSender() internal view override(Context, ERC2771Context) returns (address) {
        return ERC2771Context._msgSender();
300-301: function _msgData() internal view override(Context, ERC2771Context) returns (bytes calldata) {
        return ERC2771Context._msgData();
```

