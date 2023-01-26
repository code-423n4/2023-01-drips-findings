# GAS OPTIMIZATION REPORT
---
### Summary of optimizations
| Number | Issue details | Instances |
|---|---|:---:|
| [G-1](#G5) |`++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops | 13
| [G-2](#G6) |Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate. | 1
| [G-3](#G7) |Using storage instead of memory for structs/arrays saves gas. | 1
| [G-4](#G11) |Usage of `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead. | 206
| [G-5](#G13) |`abi.encode()` is less efficient than `abi.encodePacked()` | 4
| [G-6](#G16) |`>=` costs less gas than `>`. | 23
| [G-7](#G18) |`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 21

*Total: 7 issues.*

---
### Gas Optimizations

### <a id=G5>[G-1]</a> `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops

##### Description
In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck [cannot be made inline](https://github.com/ethereum/solidity/issues/10695). 
			Example for loop:

```Solidity
for (uint i = 0; i < length; i++) {
// do something that doesn't change the value of i
}
```

In this example, the for loop post condition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body `is 2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. You should manually do this by:

```Solidity
for (uint i = 0; i < length; i = unchecked_inc(i)) {
	// do something that doesn't change the value of i
}

function unchecked_inc(uint i) returns (uint) {
	unchecked {
		return i + 1;
	}
}
```
Or just:

```Solidity
for (uint i = 0; i < length;) {
	// do something that doesn't change the value of i
	unchecked { i++; 	}
}
```
Note that it’s important that the call to `unchecked_inc` is inlined. This is only possible for solidity versions starting from `0.8.2`.

Gas savings: roughly speaking this can save 30-40 gas per loop iteration. For lengthy loops, this can be significant!
(This is only relevant if you are using the default solidity checked arithmetic.)


##### Recommendation
Use the `unchecked` keyword

##### *Instances (13):*
File: [2023-01-drips/src/Caller.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L196 )
```solidity
196: for (uint256 i = 0; i < calls.length; i++) {
```
File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L358 )
```solidity
358: for (uint256 i = 0; i < squeezedNum; i++) {
422: for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
450: for (uint256 i = 0; i < dripsHistory.length; i++) {
563: for (uint256 i = 0; i < receivers.length; i++) {
664: for (uint256 i = 0; i < newReceivers.length; i++) {
745: for (uint256 i = 0; i < configsLen; i++) {
777: for (uint256 i = 0; i < receivers.length; i++) {
```
File: [2023-01-drips/src/DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L613 )
```solidity
613: for (uint256 i = 0; i < userMetadata.length; i++) {
```
File: [2023-01-drips/src/ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L61 )
```solidity
61: for (uint256 i = 0; i < receivers.length; i++) {
```
File: [2023-01-drips/src/Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L127 )
```solidity
127: for (uint256 i = 0; i < currReceivers.length; i++) {
158: for (uint256 i = 0; i < currReceivers.length; i++) {
231: for (uint256 i = 0; i < receivers.length; i++) {
```
### <a id=G6>[G-2]</a> Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate.

##### Description
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

##### Recommendation
Where appropriate, you can combine multiple address mappings into a single mapping of an address to a struct.

##### *Instances (1):*
File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L176 )
```solidity
176: mapping(uint256 => mapping(uint256 => DripsState)) states;
```
### <a id=G7>[G-3]</a> Using storage instead of memory for structs/arrays saves gas.

##### Description
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.
Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct([src](https://ethereum.stackexchange.com/questions/128380/why-using-storage-keyword-instead-of-memory-cost-less-gas))

##### Recommendation
Use storage for `struct/array`

##### *Instances (1):*
File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L355 )
```solidity
355: bytes32[] memory squeezedHistoryHashes = new bytes32[](squeezedNum);
```

### <a id=G11>[G-4]</a> Usage of `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.

##### Description
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

##### Recommendation
Use `uint256` instead.

##### *Instances (206):*
File: [2023-01-drips/src/AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L25 )
```solidity
25: uint32 public immutable driverId;
30: constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
60: function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
76: function give(uint256 receiver, IERC20 erc20, uint128 amt) public whenNotPaused {
128: uint32 maxEndHint1,
129: uint32 maxEndHint2,
133: _transferFromCaller(erc20, uint128(balanceDelta));
145: erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
170: function _transferFromCaller(IERC20 erc20, uint128 amt) internal {
```
File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L28 )
```solidity
28: uint32 updateTime;
30: uint32 maxEnd;
60: function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)
73: function dripId(DripsConfig config) internal pure returns (uint32) {
74: return uint32(DripsConfig.unwrap(config) >> 224);
83: function start(DripsConfig config) internal pure returns (uint32) {
84: return uint32(DripsConfig.unwrap(config) >> 32);
88: function duration(DripsConfig config) internal pure returns (uint32) {
89: return uint32(DripsConfig.unwrap(config));
108: uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
112: uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
117: uint32 internal immutable _cycleSecs;
128: /// If funds run out after the timestamp `type(uint32).max`, it's set to `type(uint32).max`.
135: uint128 balance,
136: uint32 maxEnd
153: uint256 indexed userId, uint256 indexed assetId, uint128 amt, uint32 receivableCycles
169: uint128 amt,
185: mapping(uint256 => uint32[2 ** 32]) nextSqueezed;
189: uint32 nextReceivableCycle;
191: uint32 updateTime;
193: uint32 maxEnd;
195: uint128 balance;
197: uint32 currCycleConfigs;
203: mapping(uint32 => AmtDelta) amtDeltas;
219: constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
233: function _receiveDrips(uint256 userId, uint256 assetId, uint32 maxCycles)
235: returns (uint128 receivedAmt)
237: uint32 receivableCycles;
238: uint32 fromCycle;
239: uint32 toCycle;
246: mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
247: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
270: function _receiveDripsResult(uint256 userId, uint256 assetId, uint32 maxCycles)
274: uint128 receivedAmt,
275: uint32 receivableCycles,
276: uint32 fromCycle,
277: uint32 toCycle,
287: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
289: receivedAmt += uint128(amtPerCycle);
303: returns (uint32 cycles)
305: (uint32 fromCycle, uint32 toCycle) = _receivableDripsCyclesRange(userId, assetId);
317: returns (uint32 fromCycle, uint32 toCycle)
348: ) internal returns (uint128 amt) {
357: uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
365: uint32 cycleStart = _currCycleStart();
402: uint128 amt,
419: uint32[2 ** 32] storage nextSqueezed =
421: uint32 squeezeEndCap = _currTimestamp();
425: uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
473: uint32 squeezeStartCap,
474: uint32 squeezeEndCap
475: ) private view returns (uint128 squeezedAmt) {
487: uint32 updateTime = dripsHistory.updateTime;
488: uint32 maxEnd = dripsHistory.maxEnd;
493: (uint32 start, uint32 end) =
497: return uint128(amt);
514: uint32 updateTime,
515: uint128 balance,
516: uint32 maxEnd
537: uint32 timestamp
538: ) internal view returns (uint128 balance) {
556: uint128 lastBalance,
557: uint32 lastUpdate,
558: uint32 maxEnd,
560: uint32 timestamp
561: ) private view returns (uint128 balance) {
565: (uint32 start, uint32 end) = _dripsRange({
572: balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
617: uint32 maxEndHint1,
618: uint32 maxEndHint2
623: uint32 lastUpdate = state.updateTime;
624: uint128 newBalance;
625: uint32 newMaxEnd;
627: uint32 currMaxEnd = state.maxEnd;
636: newBalance = uint128(currBalance + realBalanceDelta);
681: uint128 balance,
683: uint32 hint1,
684: uint32 hint2
685: ) private view returns (uint32 maxEnd) {
692: return uint32(enoughEnd);
695: uint256 notEnoughEnd = type(uint32).max;
697: return uint32(notEnoughEnd);
719: return uint32(end);
800: _dripsRangeInFuture(receiver, _currTimestamp(), type(uint32).max);
825: return (val >> 64, uint32(val >> 32), uint32(val));
855: uint32 updateTime,
856: uint32 maxEnd
875: uint32 lastUpdate,
876: uint32 currMaxEnd,
878: uint32 newMaxEnd
912: (uint32 currStart, uint32 currEnd) =
914: (uint32 newStart, uint32 newEnd) =
923: uint32 currStartCycle = _cycleOf(currStart);
924: uint32 newStartCycle = _cycleOf(newStart);
936: (uint32 start, uint32 end) = _dripsRangeInFuture(currRecv, lastUpdate, currMaxEnd);
944: (uint32 start, uint32 end) =
949: uint32 startCycle = _cycleOf(start);
970: function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)
973: returns (uint32 start, uint32 end)
975: return _dripsRange(receiver, updateTime, maxEnd, _currTimestamp(), type(uint32).max);
987: uint32 updateTime,
988: uint32 maxEnd,
989: uint32 startCap,
990: uint32 endCap
991: ) private pure returns (uint32 start, uint32 end_) {
997: uint40 end = uint40(start) + receiver.config.duration();
1012: return (start, uint32(end));
1020: function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)
1027: mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
1037: mapping(uint32 => AmtDelta) storage amtDeltas,
1048: AmtDelta storage amtDelta = amtDeltas[_cycleOf(uint32(timestamp))];
1120: function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
1128: function _currTimestamp() private view returns (uint32 timestamp) {
1129: return uint32(block.timestamp);
1134: function _currCycleStart() private view returns (uint32 timestamp) {
1135: uint32 currTimestamp = _currTimestamp();
```
File: [2023-01-drips/src/DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L58 )
```solidity
58: uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
64: uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
77: event DriverRegistered(uint32 indexed driverId, address indexed driverAddr);
84: uint32 indexed driverId, address indexed oldDriverAddr, address indexed newDriverAddr
97: uint32 nextDriverId;
99: mapping(uint32 => address) driverAddresses;
109: constructor(uint32 cycleSecs_)
119: uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
124: function _assertCallerIsDriver(uint32 driverId) internal view {
134: function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
145: function driverAddress(uint32 driverId) public view returns (address driverAddr) {
152: function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
160: function nextDriverId() public view returns (uint32 driverId) {
169: function cycleSecs() public view returns (uint32 cycleSecs_) {
199: returns (uint32 cycles)
216: function receiveDripsResult(uint256 userId, IERC20 erc20, uint32 maxCycles)
219: returns (uint128 receivableAmt)
238: function receiveDrips(uint256 userId, IERC20 erc20, uint32 maxCycles)
241: returns (uint128 receivedAmt)
276: ) public whenNotPaused returns (uint128 amt) {
303: ) public view returns (uint128 amt) {
317: function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
328: function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
331: returns (uint128 collectableAmt, uint128 splitAmt)
358: returns (uint128 collectableAmt, uint128 splitAmt)
372: function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
390: returns (uint128 amt)
409: function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)
438: uint32 updateTime,
439: uint128 balance,
440: uint32 maxEnd
464: uint32 timestamp
465: ) public view returns (uint128 balance) {
517: uint32 maxEndHint1,
518: uint32 maxEndHint2
521: _increaseTotalBalance(erc20, uint128(balanceDelta));
533: erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));
535: _decreaseTotalBalance(erc20, uint128(-realBalanceDelta));
536: erc20.safeTransfer(msg.sender, uint128(-realBalanceDelta));
560: uint32 updateTime,
561: uint32 maxEnd
629: function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {
635: function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {
```
File: [2023-01-drips/src/ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L15 )
```solidity
15: uint32 public immutable driverId;
17: uint32 public immutable totalSplitsWeight;
28: constructor(DripsHub _dripsHub, uint32 _driverId) {
```
File: [2023-01-drips/src/NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L25 )
```solidity
25: uint32 public immutable driverId;
32: constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
113: returns (uint128 amt)
133: function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)
193: uint32 maxEndHint1,
194: uint32 maxEndHint2,
198: _transferFromCaller(erc20, uint128(balanceDelta));
204: erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
285: function _transferFromCaller(IERC20 erc20, uint128 amt) internal {
```
File: [2023-01-drips/src/Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L11 )
```solidity
11: uint32 weight;
14: /// @notice Splits can keep track of at most `type(uint128).max`
22: uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
25: uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;
33: event Collected(uint256 indexed userId, uint256 indexed assetId, uint128 collected);
42: uint256 indexed userId, uint256 indexed receiver, uint256 indexed assetId, uint128 amt
49: event Collectable(uint256 indexed userId, uint256 indexed assetId, uint128 amt);
57: uint256 indexed userId, uint256 indexed receiver, uint256 indexed assetId, uint128 amt
71: event SplitsReceiverSeen(bytes32 indexed receiversHash, uint256 indexed userId, uint32 weight);
88: uint128 splittable;
90: uint128 collectable;
98: function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal {
106: function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
117: function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
120: returns (uint128 collectableAmt, uint128 splitAmt)
126: uint32 splitsWeight = 0;
130: splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);
145: returns (uint128 collectableAmt, uint128 splitAmt)
157: uint32 splitsWeight = 0;
160: uint128 currSplitAmt =
161: uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);
176: function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
185: function _collect(uint256 userId, uint256 assetId) internal returns (uint128 amt) {
199: function _give(uint256 userId, uint256 receiver, uint256 assetId, uint128 amt) internal {
228: uint64 totalWeight = 0;
233: uint32 weight = receiver.weight;
```

### <a id=G13>[G-5]</a> `abi.encode()` is less efficient than `abi.encodePacked()`

##### Description
`abi.encode` will apply ABI encoding rules. Therefore all elementary types are padded to 32 bytes and dynamic arrays include their length. Therefore it is possible to also decode this data again (with `abi.decode`) when the type are known.

`abi.encodePacked` will only use the only use the minimal required memory to encode the data. E.g. an address will only use 20 bytes and for dynamic arrays only the elements will be stored without length. For more info see the Solidity docs for packed mode.

For the input of the `keccak` method it is important that you can ensure that the resulting bytes of the encoding are unique. So if you always encode the same types and arrays always have the same length then there is no problem. But if you switch the parameters that you encode or encode multiple dynamic arrays you might have conflicts.
For example:
`abi.encodePacked(address(0x0000000000000000000000000000000000000001), uint(0))` encodes to the same as `abi.encodePacked(uint(0x0000000000000000000000000000000000000001000000000000000000000000), address(0))` 
and `abi.encodePacked(uint[](1,2), uint[](3))` encodes to the same as `abi.encodePacked(uint[](1), uint[](2,3))`
Therefore these examples will also generate the same hashes even so they are different inputs.
On the other hand you require less memory and therefore in most cases abi.encodePacked uses less gas than abi.encode.

##### Recommendation
Use `abi.encodePacked()` where possible to save gas

##### *Instances (4):*
File: [2023-01-drips/src/Caller.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L176 )
```solidity
176: abi.encode(
```
File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L842 )
```solidity
842: return keccak256(abi.encode(receivers));
858: return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));
```
File: [2023-01-drips/src/Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L278 )
```solidity
278: return keccak256(abi.encode(receivers));
```


### <a id=G16>[G-6]</a> `>=` costs less gas than `>`.

##### Description
The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which [saves 3 gas](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

##### Recommendation
Use `>=` instead if appropriate.

##### *Instances (23):*
File: [2023-01-drips/src/AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L132 )
```solidity
132: if (balanceDelta > 0) {
```
File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L220 )
```solidity
220: require(cycleSecs > 1, "Cycle length too low");
282: if (toCycle - fromCycle > maxCycles) {
700: if (hint1 > enoughEnd && hint1 < notEnoughEnd) {
708: if (hint2 > enoughEnd && hint2 < notEnoughEnd) {
752: if (end > maxEnd) {
756: if (spent > balance) {
779: if (i > 0) {
925: // The `currStartCycle > newStartCycle` check is just an optimization.
926: // If it's false, then `state.nextReceivableCycle > newStartCycle` must be
929: if (currStartCycle > newStartCycle && state.nextReceivableCycle > newStartCycle) {
951: if (state.nextReceivableCycle == 0 || state.nextReceivableCycle > startCycle) {
999: if (end == start || end > maxEnd) {
1005: if (end > endCap) {
```
File: [2023-01-drips/src/DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L245 )
```solidity
245: if (receivedAmt > 0) {
279: if (amt > 0) {
520: if (balanceDelta > 0) {
532: if (realBalanceDelta > 0) {
```
File: [2023-01-drips/src/ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L67 )
```solidity
67: if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);
```
File: [2023-01-drips/src/NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L69 )
```solidity
69: if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
86: if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
197: if (balanceDelta > 0) {
```
File: [2023-01-drips/src/Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L237 )
```solidity
237: if (i > 0) {
```

### <a id=G18>[G-7]</a> `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

##### Description
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

##### Recommendation
Use `<x> += <y>` instead.

##### *Instances (21):*
File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L253 )
```solidity
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
File: [2023-01-drips/src/DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L632 )
```solidity
632: totalBalances[erc20] += amt;
636: _dripsHubStorage().totalBalances[erc20] -= amt;
```
File: [2023-01-drips/src/ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L62 )
```solidity
62: weightSum += receivers[i].weight;
```
File: [2023-01-drips/src/Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L99 )
```solidity
99: _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
128: splitsWeight += currReceivers[i].weight;
159: splitsWeight += currReceivers[i].weight;
162: splitAmt += currSplitAmt;
167: collectableAmt -= splitAmt;
168: balance.collectable += collectableAmt;
235: totalWeight += weight;
```