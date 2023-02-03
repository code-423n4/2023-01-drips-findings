# Non-Critical

# [N-01] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).

```
207: return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);
 ```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

# [N-02] PRAGMA VERSION^0.8.17 VERSION TOO RECENT TO BE TRUSTED.

```
pragma solidity ^0.8.17;
```
https://github.com/ethereum/solidity/blob/develop/Changelog.md

0.8.17 (2022-09-08)
0.8.16 (2022-08-08)
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)

Unexpected bugs can be reported in recent versions;
Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs

Use a non-legacy and more battle-tested version
Use 0.8.10


# Gas Optimizations


# [GAS-01] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

#### Proof Of Concept

```solidity
247: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL247C13-L247C71

```solidity
287: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL287C8-L287C67

```solidity
358: for (uint256 i = 0; i < squeezedNum; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL358C9-L358C52

```solidity
422: for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL422C9-L422C86

```solidity
450: for (uint256 i = 0; i < dripsHistory.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL450C9-L450

```solidity
422: for (uint256 idxCap = receivers.length; idx < idxCap;) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL479C9-L479C65

```solidity
479:  for (uint256 idxCap = receivers.length; idx < idxCap;) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L479

```solidity
490: for (; idx < receivers.length; idx++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL490C9-L490C48


```solidity
563: for (uint256 i = 0; i < receivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL563C9-L563C57

```solidity
664: for (uint256 i = 0; i < newReceivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL664C13-L664C64

```solidity
745: for (uint256 i = 0; i < configsLen; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L745

```solidity
777: for (uint256 i = 0; i < receivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL777C13-L777C61

```solidity
613: for (uint256 i = 0; i < userMetadata.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#LL613C9-L613C60

```solidity
127: for (uint256 i = 0; i < currReceivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#LL127C9-L127C61

```solidity
158: for (uint256 i = 0; i < currReceivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#LL158C9-L158C61

```solidity
231: for (uint256 i = 0; i < receivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#LL231C9-L231C57

```solidity
196:  for (uint256 i = 0; i < calls.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#LL196C8-L196C53

```solidity
61:  or (uint256 i = 0; i < receivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#LL61C10-L61C57

# [G-2] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
300: function _receivableDripsCycles
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL300C5-L300C69

```
342: function _squeezeDrips
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#LL342C5-L342C28

```solidity
508: function _dripsState
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L508

```solidity
555: function _balanceAt
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L555

```
function _setDrips
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L610

Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol
```solidity
106: function _splittable
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L106

```solidity
 117: function _splitResult
 ```
 https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L117
 
 ```
 143: function _split
  ```
 
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L143

```solidity
176: function _collectable
 ```
 https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L176
 
 ```solidity
185: function _collect
  ```
  
  https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L185
  
  ```solidity
  199:function _give 
  
 ```
 
 https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L199
 
 ```
 212: function _setSplits
 ```
  
  
# [G-03] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so.

```
function driverAddress
function updateDriverAddress
function nextDriverId()function cycleSecs
function totalBalance
function receivableDripsCycles
function receiveDripsResult
function receiveDrips
function squeezeDrips
function squeezeDripsResult
function splittable
function splitResult
function split
function collectable
function collect
function give
function dripsState
function balanceAt
function setDrips
function hashDrips
function hashDripsHistory
function setSplits
function splitsHash
function hashSplits
function emitUserMetadata
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol


 ```
  function createSplits
  ```
  https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L53
 
 