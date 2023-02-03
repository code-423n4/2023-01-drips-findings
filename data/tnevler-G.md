# Report
## Gas Optimizations ##
### [G-1]: The increment in for loop post condition can be made unchecked
**Context:**

1. ```for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {``` [L247](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247) 
1. ```for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {``` [L287](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287) 
1. ```for (uint256 i = 0; i < squeezedNum; i++) {``` [L358](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L358) 
1. ```for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {``` [L422](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422) 
1. ```for (uint256 i = 0; i < dripsHistory.length; i++) {``` [L450](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450) 
1. ```for (; idx < receivers.length; idx++) {``` [L490](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490) 
1. ```for (uint256 i = 0; i < receivers.length; i++) {``` [L563](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563) 
1. ```state.currCycleConfigs++;``` [L655](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L655) 
1. ```for (uint256 i = 0; i < newReceivers.length; i++) {``` [L664](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L664) 
1. ```for (uint256 i = 0; i < configsLen; i++) {``` [L745](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L745) 
1. ```for (uint256 i = 0; i < receivers.length; i++) {``` [L777](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L777) 
1. ```currIdx++;``` [L959](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L959) 
1. ```newIdx++;``` [L962](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L962) 
1. ```driverId = dripsHubStorage.nextDriverId++;``` [L136](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L136) 
1. ```for (uint256 i = 0; i < userMetadata.length; i++) {``` [L613](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613) 
1. ```for (uint256 i = 0; i < currReceivers.length; i++) {``` [L127](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127) 
1. ```for (uint256 i = 0; i < currReceivers.length; i++) {``` [L158](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158) 
1. ```for (uint256 i = 0; i < receivers.length; i++) {``` [L231](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231) 
1. ```StorageSlot.getUint256Slot(_mintedTokensSlot).value++;``` [L93](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L93) 
1. ```uint256 currNonce = nonce[sender]++;``` [L174](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L174) 
1. ```for (uint256 i = 0; i < calls.length; i++) {``` [L196](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196) 
1. ```StorageSlot.getUint256Slot(_counterSlot).value++;``` [L59](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L59) 
1. ```for (uint256 i = 0; i < receivers.length; i++) {``` [L61](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61) 

**Description:**

[This](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked) can save 30-40 gas per loop iteration.

**Recommendation:**

Example how to fix. Change:
```
for (uint256 i = 0; i < orders.length; ++i) {
   // Do the thing
}
```

To:
```
for (uint256 i = 0; i < orders.length;) {
   // Do the thing
   unchecked { ++i; }
}
```

### [G-2]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```receivableCycles = toCycle - fromCycle - maxCycles;``` [L283](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283) 
1. ```uint256 revIdx = squeezedRevIdxs[squeezedNum - i - 1];``` [L361](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L361) 
1. ```DripsHistory memory drips = dripsHistory[dripsHistory.length - i];``` [L423](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L423) 
1. ```uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];``` [L425](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L425) 
1. ```require(_isOrdered(receivers[i - 1], receiver), "Receivers not sorted");``` [L780](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L780) 

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.