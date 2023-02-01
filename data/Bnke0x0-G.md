


### [G01] `++i/i++` should be `unchecked{++i}`/`unchecked{++i}` when it is not possible for them to overflow, as is the case when used in `for` and `while` loops


#### Findings:
```
2023-01-drips/src/Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {
2023-01-drips/src/Drips.sol::247 => for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
2023-01-drips/src/Drips.sol::287 => for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
2023-01-drips/src/Drips.sol::358 => for (uint256 i = 0; i < squeezedNum; i++) {
2023-01-drips/src/Drips.sol::422 => for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
2023-01-drips/src/Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips/src/Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips/src/Drips.sol::745 => for (uint256 i = 0; i < configsLen; i++) {
2023-01-drips/src/Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {
2023-01-drips/src/ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
```







### [G02] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

#### Impact
> When using elements that are smaller than 32 bytes, your 
contract’s gas usage may be higher. This is because the EVM operates on 
32 bytes at a time. Therefore, if the element is smaller than that, the 
EVM must use more operations in order to reduce the size of the element 
from 32 bytes to the desired size.
> 

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)
Use a larger size then downcast where needed
#### Findings:
```
2023-01-drips/src/AddressDriver.sol::25 => uint32 public immutable driverId;
2023-01-drips/src/AddressDriver.sol::30 => constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
2023-01-drips/src/AddressDriver.sol::60 => function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
2023-01-drips/src/AddressDriver.sol::128 => uint32 maxEndHint1,
2023-01-drips/src/AddressDriver.sol::129 => uint32 maxEndHint2,
2023-01-drips/src/AddressDriver.sol::133 => _transferFromCaller(erc20, uint128(balanceDelta));
2023-01-drips/src/AddressDriver.sol::145 => erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
2023-01-drips/src/AddressDriver.sol::170 => function _transferFromCaller(IERC20 erc20, uint128 amt) internal {
2023-01-drips/src/Drips.sol::28 => uint32 updateTime;
2023-01-drips/src/Drips.sol::30 => uint32 maxEnd;
2023-01-drips/src/Drips.sol::60 => function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)
2023-01-drips/src/Drips.sol::73 => function dripId(DripsConfig config) internal pure returns (uint32) {
2023-01-drips/src/Drips.sol::74 => return uint32(DripsConfig.unwrap(config) >> 224);
2023-01-drips/src/Drips.sol::78 => function amtPerSec(DripsConfig config) internal pure returns (uint160) {
2023-01-drips/src/Drips.sol::79 => return uint160(DripsConfig.unwrap(config) >> 64);
2023-01-drips/src/Drips.sol::83 => function start(DripsConfig config) internal pure returns (uint32) {
2023-01-drips/src/Drips.sol::84 => return uint32(DripsConfig.unwrap(config) >> 32);
2023-01-drips/src/Drips.sol::88 => function duration(DripsConfig config) internal pure returns (uint32) {
2023-01-drips/src/Drips.sol::89 => return uint32(DripsConfig.unwrap(config));
2023-01-drips/src/Drips.sol::108 => uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
2023-01-drips/src/Drips.sol::117 => uint32 internal immutable _cycleSecs;
2023-01-drips/src/Drips.sol::135 => uint128 balance,
2023-01-drips/src/Drips.sol::136 => uint32 maxEnd
2023-01-drips/src/Drips.sol::169 => uint128 amt,
2023-01-drips/src/Drips.sol::189 => uint32 nextReceivableCycle;
2023-01-drips/src/Drips.sol::191 => uint32 updateTime;
2023-01-drips/src/Drips.sol::193 => uint32 maxEnd;
2023-01-drips/src/Drips.sol::195 => uint128 balance;
2023-01-drips/src/Drips.sol::197 => uint32 currCycleConfigs;
2023-01-drips/src/Drips.sol::203 => mapping(uint32 => AmtDelta) amtDeltas;
2023-01-drips/src/Drips.sol::219 => constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
2023-01-drips/src/Drips.sol::235 => returns (uint128 receivedAmt)
2023-01-drips/src/Drips.sol::237 => uint32 receivableCycles;
2023-01-drips/src/Drips.sol::238 => uint32 fromCycle;
2023-01-drips/src/Drips.sol::239 => uint32 toCycle;
2023-01-drips/src/Drips.sol::246 => mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
2023-01-drips/src/Drips.sol::247 => for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
2023-01-drips/src/Drips.sol::274 => uint128 receivedAmt,
2023-01-drips/src/Drips.sol::275 => uint32 receivableCycles,
2023-01-drips/src/Drips.sol::276 => uint32 fromCycle,
2023-01-drips/src/Drips.sol::277 => uint32 toCycle,
2023-01-drips/src/Drips.sol::287 => for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
2023-01-drips/src/Drips.sol::289 => receivedAmt += uint128(amtPerCycle);
2023-01-drips/src/Drips.sol::303 => returns (uint32 cycles)
2023-01-drips/src/Drips.sol::305 => (uint32 fromCycle, uint32 toCycle) = _receivableDripsCyclesRange(userId, assetId);
2023-01-drips/src/Drips.sol::317 => returns (uint32 fromCycle, uint32 toCycle)
2023-01-drips/src/Drips.sol::348 => ) internal returns (uint128 amt) {
2023-01-drips/src/Drips.sol::357 => uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
2023-01-drips/src/Drips.sol::365 => uint32 cycleStart = _currCycleStart();
2023-01-drips/src/Drips.sol::402 => uint128 amt,
2023-01-drips/src/Drips.sol::419 => uint32[2 ** 32] storage nextSqueezed =
2023-01-drips/src/Drips.sol::421 => uint32 squeezeEndCap = _currTimestamp();
2023-01-drips/src/Drips.sol::425 => uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
2023-01-drips/src/Drips.sol::473 => uint32 squeezeStartCap,
2023-01-drips/src/Drips.sol::474 => uint32 squeezeEndCap
2023-01-drips/src/Drips.sol::475 => ) private view returns (uint128 squeezedAmt) {
2023-01-drips/src/Drips.sol::487 => uint32 updateTime = dripsHistory.updateTime;
2023-01-drips/src/Drips.sol::488 => uint32 maxEnd = dripsHistory.maxEnd;
2023-01-drips/src/Drips.sol::493 => (uint32 start, uint32 end) =
2023-01-drips/src/Drips.sol::497 => return uint128(amt);
2023-01-drips/src/Drips.sol::514 => uint32 updateTime,
2023-01-drips/src/Drips.sol::515 => uint128 balance,
2023-01-drips/src/Drips.sol::516 => uint32 maxEnd
2023-01-drips/src/Drips.sol::537 => uint32 timestamp
2023-01-drips/src/Drips.sol::538 => ) internal view returns (uint128 balance) {
2023-01-drips/src/Drips.sol::556 => uint128 lastBalance,
2023-01-drips/src/Drips.sol::557 => uint32 lastUpdate,
2023-01-drips/src/Drips.sol::558 => uint32 maxEnd,
2023-01-drips/src/Drips.sol::560 => uint32 timestamp
2023-01-drips/src/Drips.sol::561 => ) private view returns (uint128 balance) {
2023-01-drips/src/Drips.sol::565 => (uint32 start, uint32 end) = _dripsRange({
2023-01-drips/src/Drips.sol::572 => balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
2023-01-drips/src/Drips.sol::617 => uint32 maxEndHint1,
2023-01-drips/src/Drips.sol::618 => uint32 maxEndHint2
2023-01-drips/src/Drips.sol::623 => uint32 lastUpdate = state.updateTime;
2023-01-drips/src/Drips.sol::624 => uint128 newBalance;
2023-01-drips/src/Drips.sol::625 => uint32 newMaxEnd;
2023-01-drips/src/Drips.sol::627 => uint32 currMaxEnd = state.maxEnd;
2023-01-drips/src/Drips.sol::636 => newBalance = uint128(currBalance + realBalanceDelta);
2023-01-drips/src/Drips.sol::681 => uint128 balance,
2023-01-drips/src/Drips.sol::683 => uint32 hint1,
2023-01-drips/src/Drips.sol::684 => uint32 hint2
2023-01-drips/src/Drips.sol::685 => ) private view returns (uint32 maxEnd) {
2023-01-drips/src/Drips.sol::692 => return uint32(enoughEnd);
2023-01-drips/src/Drips.sol::697 => return uint32(notEnoughEnd);
2023-01-drips/src/Drips.sol::719 => return uint32(end);
2023-01-drips/src/Drips.sol::800 => _dripsRangeInFuture(receiver, _currTimestamp(), type(uint32).max);
2023-01-drips/src/Drips.sol::825 => return (val >> 64, uint32(val >> 32), uint32(val));
2023-01-drips/src/Drips.sol::855 => uint32 updateTime,
2023-01-drips/src/Drips.sol::856 => uint32 maxEnd
2023-01-drips/src/Drips.sol::875 => uint32 lastUpdate,
2023-01-drips/src/Drips.sol::876 => uint32 currMaxEnd,
2023-01-drips/src/Drips.sol::878 => uint32 newMaxEnd
2023-01-drips/src/Drips.sol::912 => (uint32 currStart, uint32 currEnd) =
2023-01-drips/src/Drips.sol::914 => (uint32 newStart, uint32 newEnd) =
2023-01-drips/src/Drips.sol::923 => uint32 currStartCycle = _cycleOf(currStart);
2023-01-drips/src/Drips.sol::924 => uint32 newStartCycle = _cycleOf(newStart);
2023-01-drips/src/Drips.sol::936 => (uint32 start, uint32 end) = _dripsRangeInFuture(currRecv, lastUpdate, currMaxEnd);
2023-01-drips/src/Drips.sol::944 => (uint32 start, uint32 end) =
2023-01-drips/src/Drips.sol::949 => uint32 startCycle = _cycleOf(start);
2023-01-drips/src/Drips.sol::970 => function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)
2023-01-drips/src/Drips.sol::973 => returns (uint32 start, uint32 end)
2023-01-drips/src/Drips.sol::975 => return _dripsRange(receiver, updateTime, maxEnd, _currTimestamp(), type(uint32).max);
2023-01-drips/src/Drips.sol::987 => uint32 updateTime,
2023-01-drips/src/Drips.sol::988 => uint32 maxEnd,
2023-01-drips/src/Drips.sol::989 => uint32 startCap,
2023-01-drips/src/Drips.sol::990 => uint32 endCap
2023-01-drips/src/Drips.sol::991 => ) private pure returns (uint32 start, uint32 end_) {
2023-01-drips/src/Drips.sol::997 => uint40 end = uint40(start) + receiver.config.duration();
2023-01-drips/src/Drips.sol::1012 => return (start, uint32(end));
2023-01-drips/src/Drips.sol::1020 => function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)
2023-01-drips/src/Drips.sol::1027 => mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
2023-01-drips/src/Drips.sol::1037 => mapping(uint32 => AmtDelta) storage amtDeltas,p) private view returns (uint32 cycle) {
2023-01-drips/src/Drips.sol::1128 => function _currTimestamp() private view returns (uint32 timestamp) {
2023-01-drips/src/Drips.sol::1129 => return uint32(block.timestamp);
2023-01-drips/src/Drips.sol::1134 => function _currCycleStart() private view returns (uint32 timestamp) {
2023-01-drips/src/Drips.sol::1135 => uint32 currTimestamp = _currTimestamp();
2023-01-drips/src/DripsHub.sol::58 => uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
2023-01-drips/src/DripsHub.sol::64 => uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
2023-01-drips/src/DripsHub.sol::77 => event DriverRegistered(uint32 indexed driverId, address indexed driverAddr);
2023-01-drips/src/DripsHub.sol::84 => uint32 indexed driverId, address indexed oldDriverAddr, address indexed newDriverAddr
2023-01-drips/src/DripsHub.sol::97 => uint32 nextDriverId;
2023-01-drips/src/DripsHub.sol::99 => mapping(uint32 => address) driverAddresses;
2023-01-drips/src/DripsHub.sol::109 => constructor(uint32 cycleSecs_)
2023-01-drips/src/DripsHub.sol::119 => uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
2023-01-drips/src/DripsHub.sol::124 => function _assertCallerIsDriver(uint32 driverId) internal view {
2023-01-drips/src/DripsHub.sol::134 => function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
2023-01-drips/src/DripsHub.sol::145 => function driverAddress(uint32 driverId) public view returns (address driverAddr) {
2023-01-drips/src/DripsHub.sol::152 => function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
2023-01-drips/src/DripsHub.sol::160 => function nextDriverId() public view returns (uint32 driverId) {
2023-01-drips/src/DripsHub.sol::169 => function cycleSecs() public view returns (uint32 cycleSecs_) {
2023-01-drips/src/DripsHub.sol::199 => returns (uint32 cycles)
2023-01-drips/src/DripsHub.sol::219 => returns (uint128 receivableAmt)
2023-01-drips/src/DripsHub.sol::241 => returns (uint128 receivedAmt)
2023-01-drips/src/DripsHub.sol::276 => ) public whenNotPaused returns (uint128 amt) {
2023-01-drips/src/DripsHub.sol::303 => ) public view returns (uint128 amt) {
2023-01-drips/src/DripsHub.sol::331 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/DripsHub.sol::358 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/DripsHub.sol::390 => returns (uint128 amt)
2023-01-drips/src/DripsHub.sol::438 => uint32 updateTime,
2023-01-drips/src/DripsHub.sol::439 => uint128 balance,
2023-01-drips/src/DripsHub.sol::440 => uint32 maxEnd
2023-01-drips/src/DripsHub.sol::464 => uint32 timestamp
2023-01-drips/src/DripsHub.sol::465 => ) public view returns (uint128 balance) {
2023-01-drips/src/DripsHub.sol::517 => uint32 maxEndHint1,
2023-01-drips/src/DripsHub.sol::518 => uint32 maxEndHint2
2023-01-drips/src/DripsHub.sol::521 => _increaseTotalBalance(erc20, uint128(balanceDelta));
2023-01-drips/src/DripsHub.sol::533 => erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));
2023-01-drips/src/DripsHub.sol::535 => _decreaseTotalBalance(erc20, uint128(-realBalanceDelta));
2023-01-drips/src/DripsHub.sol::536 => erc20.safeTransfer(msg.sender, uint128(-realBalanceDelta));
2023-01-drips/src/DripsHub.sol::560 => uint32 updateTime,
2023-01-drips/src/DripsHub.sol::561 => uint32 maxEnd
2023-01-drips/src/DripsHub.sol::629 => function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {
2023-01-drips/src/DripsHub.sol::635 => function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {
2023-01-drips/src/DripsHub.sol::643 => return uint160(address(erc20));
2023-01-drips/src/ImmutableSplitsDriver.sol::15 => uint32 public immutable driverId;
2023-01-drips/src/ImmutableSplitsDriver.sol::17 => uint32 public immutable totalSplitsWeight;
2023-01-drips/src/ImmutableSplitsDriver.sol::28 => constructor(DripsHub _dripsHub, uint32 _driverId) {
2023-01-drips/src/NFTDriver.sol::25 => uint32 public immutable driverId;
2023-01-drips/src/NFTDriver.sol::32 => constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
2023-01-drips/src/NFTDriver.sol::193 => uint32 maxEndHint1,
2023-01-drips/src/NFTDriver.sol::194 => uint32 maxEndHint2,
2023-01-drips/src/NFTDriver.sol::198 => _transferFromCaller(erc20, uint128(balanceDelta));
2023-01-drips/src/NFTDriver.sol::204 => erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
2023-01-drips/src/Splits.sol::11 => uint32 weight;
2023-01-drips/src/Splits.sol::22 => uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
2023-01-drips/src/Splits.sol::88 => uint128 splittable;
2023-01-drips/src/Splits.sol::90 => uint128 collectable;
2023-01-drips/src/Splits.sol::120 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/Splits.sol::126 => uint32 splitsWeight = 0;
2023-01-drips/src/Splits.sol::130 => splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);
2023-01-drips/src/Splits.sol::145 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/Splits.sol::157 => uint32 splitsWeight = 0;
2023-01-drips/src/Splits.sol::160 => uint128 currSplitAmt =
2023-01-drips/src/Splits.sol::161 => uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);
2023-01-drips/src/Splits.sol::228 => uint64 totalWeight = 0;
2023-01-drips/src/Splits.sol::233 => uint32 weight = receiver.weight;
```








### [G03] Use `calldata` instead of `memory` for function parameters

#### Impact
Use calldata instead of memory for function parameters. Having function arguments use calldata instead of memory can save gas.
#### Findings:
```
2023-01-drips/src/Managed.sol::136 => function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
```
### Recommended Mitigation Steps

Change function arguments from memory to calldata.



### [G04] Amounts should be checked for 0 before calling a transfer

#### Impact
Checking non-zero transfer values can avoid an expensive external call and save gas.

While this is done in some places, it’s not consistently done in the solution.
#### Findings:
```
2023-01-drips/src/AddressDriver.sol::62 => erc20.safeTransfer(transferTo, amt);
2023-01-drips/src/AddressDriver.sol::145 => erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
2023-01-drips/src/DripsHub.sol::394 => erc20.safeTransfer(msg.sender, amt);
2023-01-drips/src/DripsHub.sol::536 => erc20.safeTransfer(msg.sender, uint128(-realBalanceDelta));
2023-01-drips/src/NFTDriver.sol::116 => erc20.safeTransfer(transferTo, amt);
2023-01-drips/src/NFTDriver.sol::204 => erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
2023-01-drips/src/NFTDriver.sol::282 => super.transferFrom(from, to, tokenId);
```



### [G05] Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate

#### Impact
Saves a storage slot for the mapping. Depending on the circumstances 
and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. 
Reads and subsequent writes can also be cheaper when a function requires
 both values and they both fit in the same storage slot
#### Findings:
```
2023-01-drips/src/Drips.sol::203 => mapping(uint32 => AmtDelta) amtDeltas;
2023-01-drips/src/Drips.sol::246 => mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
2023-01-drips/src/Drips.sol::1027 => mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
2023-01-drips/src/Drips.sol::1037 => mapping(uint32 => AmtDelta) storage amtDeltas,
2023-01-drips/src/Splits.sol::76 => mapping(uint256 => SplitsState) splitsStates;
2023-01-drips/src/Splits.sol::83 => mapping(uint256 => SplitsBalance) balances;
```







### [G06] `abi.encode()` is less efficient than abi.encodePacked()



#### Findings:
```
2023-01-drips/src/Caller.sol::176 => abi.encode(
2023-01-drips/src/Drips.sol::842 => return keccak256(abi.encode(receivers));
2023-01-drips/src/Drips.sol::858 => return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));
2023-01-drips/src/Splits.sol::278 => return keccak256(abi.encode(receivers));
```


