## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW | 24 |
| [GAS-2](#GAS-2) | X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES | 21 |
| [GAS-3](#GAS-3) | USING BOOLS FOR STORAGE INCURS OVERHEAD | 1 |
| [GAS-4](#GAS-4) | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS | 49 |
| [GAS-5](#GAS-5) | USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD | 14 |
| [GAS-6](#GAS-6) | STATE VARIABLES CAN BE PACKED INTO FEWER STORAGE SLOTS | 2 |
| [GAS-7](#GAS-7) | USE CONSTANT INSTEAD OF IMMUTABLE FOR DECLARED STATE VARIABLES | 5 |

### [GAS-1] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.

*Instances (24):*
```solidity
File: Caller.sol

174:      uint256 currNonce = nonce[sender]++;

196:      for (uint256 i = 0; i < calls.length; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol)

```solidity
File: Drips.sol

247:      for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

287:      for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

358:      for (uint256 i = 0; i < squeezedNum; i++) {

422:      for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

428:      squeezedRevIdxs[squeezedNum++] = i;

450:      for (uint256 i = 0; i < dripsHistory.length; i++) {

490:      for (; idx < receivers.length; idx++) {

563:      for (uint256 i = 0; i < receivers.length; i++) {

655:      state.currCycleConfigs++;

664:      for (uint256 i = 0; i < newReceivers.length; i++) {

745:      for (uint256 i = 0; i < configsLen; i++) {

777:      for (uint256 i = 0; i < receivers.length; i++) {

959:      currIdx++;

962:      newIdx++;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)

```solidity
File: DripsHub.sol

136:      driverId = dripsHubStorage.nextDriverId++;

613:      for (uint256 i = 0; i < userMetadata.length; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)

```solidity
File: ImmutableSplitsDriver.sol

59:      StorageSlot.getUint256Slot(_counterSlot).value++;

61:      for (uint256 i = 0; i < receivers.length; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol)


```solidity
File: NFTDriver.sol

93:      StorageSlot.getUint256Slot(_mintedTokensSlot).value++;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L93)

```solidity
File: Splits.sol

127:      for (uint256 i = 0; i < currReceivers.length; i++) {

158:      for (uint256 i = 0; i < currReceivers.length; i++) {

231:      for (uint256 i = 0; i < receivers.length; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol)

### [GAS-2] X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES

*Instances (21):*
```solidity
File: Drips.sol

253:      amtDeltas[toCycle].thisCycle += finalAmtPerCycle;

284:      toCycle -= receivableCycles;

288:      amtPerCycle += state.amtDeltas[cycle].thisCycle;
289:      receivedAmt += uint128(amtPerCycle);
290:      amtPerCycle += state.amtDeltas[cycle].nextCycle;

429:      amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);

495:      amt += _drippedAmt(receiver.config.amtPerSec(), start, end);

572:      balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

755:      spent += _drippedAmt(amtPerSec, start, end);

1053:     amtDelta.thisCycle += int128(fullCycle - nextCycle);
1054:     amtDelta.nextCycle += int128(nextCycle);


```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)

```solidity
File: DripsHub.sol

632:      totalBalances[erc20] += amt;

636:      _dripsHubStorage().totalBalances[erc20] -= amt;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632)

```solidity
File: ImmutableSplitsDriver.sol

62:      weightSum += receivers[i].weight;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L62)

```solidity
File: Splits.sol

99:       _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;

128:      splitsWeight += currReceivers[i].weight;

159:      splitsWeight += currReceivers[i].weight;

162:      splitAmt += currSplitAmt;

167:      collectableAmt -= splitAmt;
168:      balance.collectable += collectableAmt;

235:      totalWeight += weight;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol)

### [GAS-3] USING BOOLS FOR STORAGE INCURS OVERHEAD

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from `false` to `true`, after having been `true` in the past.

*Instance (1):*
```solidity
File: Managed.sol

41:     bool isPaused;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol)

### [GAS-4] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

*Instances (49):*
```solidity
File: AddressDriver.sol

40:     function calcUserId(address userAddr) public view returns (uint256 userId) {

46:     function callerUserId() internal view returns (uint256 userId) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol)

```solidity
File: Caller.sol

124:      function isAuthorized(address sender, address user) public view returns (bool authorized) {

132:      function allAuthorized(address sender) public view returns (address[] memory authorized) {

147:      returns (bytes memory returnData)

171:      ) public payable returns (bytes memory returnData) {

204:      returns (bytes memory returnData)

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol)

```solidity
File: Drips.sol

303:      returns (uint32 cycles)

475:      ) private view returns (uint128 squeezedAmt) {

511:      returns (

538:      ) internal view returns (uint128 balance) {

685:      ) private view returns (uint32 maxEnd) {

742:      ) private view returns (bool isEnough) {

795:      returns (uint256 newConfigsLen)

818:      returns (uint256 amtPerSec, uint256 start, uint256 end)

837:      returns (bytes32 dripsHash)

857:      ) internal pure returns (bytes32 dripsHistoryHash) {

973:      returns (uint32 start, uint32 end)

991:      ) private pure returns (uint32 start, uint32 end_) {

1120:     function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {

1128:     function _currTimestamp() private view returns (uint32 timestamp) {

1134:     function _currCycleStart() private view returns (uint32 timestamp) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)

```solidity
File: DripsHub.sol

145:     function driverAddress(uint32 driverId) public view returns (address driverAddr) {

160:     function nextDriverId() public view returns (uint32 driverId) {

169:     function cycleSecs() public view returns (uint32 cycleSecs_) {

181:     function totalBalance(IERC20 erc20) public view returns (uint256 balance) {

199:     returns (uint32 cycles)

317:     function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

331:     returns (uint128 collectableAmt, uint128 splitAmt)

358:     returns (uint128 collectableAmt, uint128 splitAmt)

372:     function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

435:     returns (

465:     ) public view returns (uint128 balance) {

546:     function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {

562:     ) public pure returns (bytes32 dripsHistoryHash) {

587:     function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {

598:     returns (bytes32 receiversHash)

642:     function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)

```solidity
File: ImmutableSplitsDriver.sol

36:      function nextUserId() public view returns (uint256 userId) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L36)

```solidity
File: Managed.sol

105:      function isPauser(address pauser) public view returns (bool isAddrPauser) {

112:      function allPausers() public view returns (address[] memory pausersList) {

117:      function paused() public view returns (bool isPaused) {      

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol)

```solidity
File: NFTDriver.sol

50:      function nextTokenId() public view returns (uint256 tokenId) {

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L50)

```solidity
File: Splits.sol

106:      function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {      

120:      returns (uint128 collectableAmt, uint128 splitAmt)

145:      returns (uint128 collectableAmt, uint128 splitAmt)

176:      function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {

262:      function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {

273:      returns (bytes32 receiversHash)

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol)

### [GAS-5] USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

*Instances (14):*
```solidity
File: AddressDriver.sol

25:     uint32 public immutable driverId;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L25)

```solidity
File: Drips.sol

28:     uint32 updateTime;

30:     uint32 maxEnd;

135:    uint128 balance,
136:    uint32 maxEnd

185:    mapping(uint256 => uint32[2 ** 32]) nextSqueezed;

203:    mapping(uint32 => AmtDelta) amtDeltas;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)

```solidity
File: DripsHub.sol

97:     uint32 nextDriverId;

99:     mapping(uint32 => address) driverAddresses;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)

```solidity
File: ImmutableSplitsDriver.sol

15:     uint32 public immutable driverId;

17:     uint32 public immutable totalSplitsWeight;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol)

```solidity
File: NFTDriver.sol

15:     uint32 public immutable driverId;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol)

```solidity
File: Splits.sol

11:     uint32 weight;

22:     uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol)

### [GAS-6] STATE VARIABLES CAN BE PACKED INTO FEWER STORAGE SLOTS

If variables occupying the same slot are both written the same function or by the constructor, it avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper. Consider making this state var near one of the address types (doesn't matter which because its not used in the same functions)

*Instance (2):*

Here `_AMT_PER_SEC_EXTRA_DECIMALS` can be packed with `_cycleSecs` in a single Slot.

```solidity
File: Drips.sol

      /// @notice The additional decimals for all amtPerSec values.
    uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
    /// @notice The multiplier for all amtPerSec values. It's `10 ** _AMT_PER_SEC_EXTRA_DECIMALS`.
    uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
    /// @notice The total amount the contract can keep track of each asset.
    uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
    /// @notice On every timestamp `T`, which is a multiple of `cycleSecs`, the receivers
    /// gain access to drips received during `T - cycleSecs` to `T - 1`.
    /// Always higher than 1.
    // slither-disable-next-line naming-convention
    uint32 internal immutable _cycleSecs;

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)

Here `AMT_PER_SEC_EXTRA_DECIMALS` can be packed with `TOTAL_SPLITS_WEIGHT` in a single Slot.

```solidity
File: DripsHub.sol

    /// @notice The additional decimals for all amtPerSec values.
    uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
    /// @notice The multiplier for all amtPerSec values.
    uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
    /// @notice Maximum number of splits receivers of a single user. Limits the cost of splitting.
    uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
    /// @notice The total splits weight of a user
    uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;    

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L57-L64)

### [GAS-7] USE CONSTANT INSTEAD OF IMMUTABLE FOR DECLARED STATE VARIABLES

According to Solidity Docs:
>State variables can be declared as constant or immutable. In both cases, the variables cannot be modified after the contract has been constructed. For constant variables, the value has to be fixed at compile-time, while for immutable, it can still be assigned at construction time.

Constant is cheaper than immutable.

*Instance (5):*
```solidity
File: Caller.sol

84:    bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L84)

```solidity
File: DripsHub.sol

72:    bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage");

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L72)

```solidity
File: ImmutableSplitsDriver.sol

19:    bytes32 private immutable _counterSlot = _erc1967Slot("eip1967.immutableSplitsDriver.storage");

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L19)

```solidity
File: Managed.sol

20:    bytes32 private immutable _managedStorageSlot = _erc1967Slot("eip1967.managed.storage");

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L20)

```solidity
File: NFTDriver.sol

27:    bytes32 private immutable _mintedTokensSlot = _erc1967Slot("eip1967.nftDriver.storage");

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L27)