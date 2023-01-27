## [G-01] Using bools for storage incurs overhead

### Description

Use uint256 for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

### Findings

- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol
  ```Solidity
  ::108 =>     uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
  ::185 =>         mapping(uint256 => uint32[2 ** 32]) nextSqueezed;
  ::203 =>         mapping(uint32 => AmtDelta) amtDeltas;
  ::246 =>             mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
  ::247 =>             for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
  ::287 =>         for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
  ::305 =>         (uint32 fromCycle, uint32 toCycle) = _receivableDripsCyclesRange(userId, assetId);
  ::357 =>         uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
  ::365 =>         uint32 cycleStart = _currCycleStart();
  ::419 =>         uint32[2 ** 32] storage nextSqueezed =
  ::421 =>         uint32 squeezeEndCap = _currTimestamp();
  ::425 =>                 uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
  ::487 =>         uint32 updateTime = dripsHistory.updateTime;
  ::488 =>         uint32 maxEnd = dripsHistory.maxEnd;
  ::493 =>             (uint32 start, uint32 end) =
  ::565 =>             (uint32 start, uint32 end) = _dripsRange({
  ::623 =>         uint32 lastUpdate = state.updateTime;
  ::627 =>             uint32 currMaxEnd = state.maxEnd;
  ::695 =>             uint256 notEnoughEnd = type(uint32).max;
  ::912 =>                 (uint32 currStart, uint32 currEnd) =
  ::914 =>                 (uint32 newStart, uint32 newEnd) =
  ::923 =>                 uint32 currStartCycle = _cycleOf(currStart);
  ::924 =>                 uint32 newStartCycle = _cycleOf(newStart);
  ::936 =>                 (uint32 start, uint32 end) = _dripsRangeInFuture(currRecv, lastUpdate, currMaxEnd);
  ::944 =>                 (uint32 start, uint32 end) =
  ::949 =>                 uint32 startCycle = _cycleOf(start);
  ::997 =>         uint40 end = uint40(start) + receiver.config.duration();
  ::1027 =>         mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
  ::1037 =>         mapping(uint32 => AmtDelta) storage amtDeltas,
  ::1048 =>             AmtDelta storage amtDelta = amtDeltas[_cycleOf(uint32(timestamp))];
  ::1135 =>         uint32 currTimestamp = _currTimestamp();
  ```
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol
  ```Solidity
  ::58 =>     uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
  ::64 =>     uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
  ::99 =>         mapping(uint32 => address) driverAddresses;
  ::119 =>         uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
  ```
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol
  ```Solidity
  ::22 =>     uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
  ::126 =>         uint32 splitsWeight = 0;
  ::157 =>         uint32 splitsWeight = 0;
  ::228 =>         uint64 totalWeight = 0;
  ::233 =>             uint32 weight = receiver.weight;
  ```

### References

- [[GAS-1] Using bools for storage incurs overhead](https://gist.github.com/Picodes/ab2df52379e4b4993709be1b91aab651#gas-1-using-bools-for-storage-incurs-overhead)
- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27)


## [G-02] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

### Description

Gas consumption can be greater if you use items that are less than 32 bytes in size. This is such that the EVM can only handle 32 bytes at once. In order to increase the element's size from 32 bytes to the necessary amount, the EVM must do extra operations if it is lower than that. When necessary, it is advised to utilize a bigger size and then downcast.

### Findings

- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol
  ```Solidity
  ::883 =>             bool pickCurr = currIdx < currReceivers.length;
  ::890 =>             bool pickNew = newIdx < newReceivers.length;
  ```
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol
  ```Solidity
  ::41 =>         bool isPaused;
  ```

### References

- [Layout of State Variables in Storage | Solidity docs](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html#layout-of-state-variables-in-storage)
- [GAS OPTIMIZATIONS ISSUES by Gokberk Gulgun](https://hackmd.io/@W1m6lTsFT5WAy9C_lRTX_g/rkr5Laoys)