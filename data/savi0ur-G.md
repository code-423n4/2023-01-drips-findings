# [G-01] ++i/i++ should be unchecked{ ++i } / unchecked{ i++ } when its not posssible for them to overflow, as its in the case when used in for loops/while loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

## Proof Of Concept
```solidity
196: for (uint256 i = 0; i < calls.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196

```solidity
247: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247

```solidity
287: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287

```solidity
358: for (uint256 i = 0; i < squeezedNum; i++)
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L358

```solidity
422: for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422

```solidity
450: for (uint256 i = 0; i < dripsHistory.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450

```solidity
490: for (; idx < receivers.length; idx++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490

```solidity
563: for (uint256 i = 0; i < receivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563

```solidity
664: for (uint256 i = 0; i < newReceivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L664

```solidity
745: for (uint256 i = 0; i < configsLen; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L745

```solidity
777: for (uint256 i = 0; i < receivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L777

```solidity
613: for (uint256 i = 0; i < userMetadata.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613

```solidity
61: for (uint256 i = 0; i < receivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61

```solidity
127: for (uint256 i = 0; i < currReceivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127

```solidity
158: for (uint256 i = 0; i < currReceivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158

```solidity
231: for (uint256 i = 0; i < receivers.length; i++) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231

# [G-02] Cache storage values in memory to minimize SLOADS

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.
## Proof Of Concept
```solidity
DripsState storage state = _dripsStorage().states[assetId][userId]; //@audit use memory instead of storage
for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
    amtPerCycle += state.amtDeltas[cycle].thisCycle; //@audit SLOAD
    receivedAmt += uint128(amtPerCycle);
    amtPerCycle += state.amtDeltas[cycle].nextCycle; //@audit SLOAD
}
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L286-L291

```solidity
DripsState storage sender = _dripsStorage().states[assetId][senderId]; //@audit use memory instead of storage
historyHashes = _verifyDripsHistory(historyHash, dripsHistory, sender.dripsHistoryHash); //@audit SLOAD
// If the last update was not in the current cycle,
// there's only the single latest history entry to squeeze in the current cycle.
currCycleConfigs = 1;
// slither-disable-next-line timestamp
if (sender.updateTime >= _currCycleStart()) currCycleConfigs = sender.currCycleConfigs; //@audit 2 SLOADS
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L410-L416

```solidity
DripsState storage state = _dripsStorage().states[assetId][userId]; //@audit use memory instead of storage
return
    (state.dripsHash, state.dripsHistoryHash, state.updateTime, state.balance, state.maxEnd); //@audit 5 SLOADS
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L519-L521

```solidity
DripsState storage state = _dripsStorage().states[assetId][userId]; //@audit use memory instead of storage
require(timestamp >= state.updateTime, "Timestamp before last drips update"); //@audit SLOAD
require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list"); //@audit SLOAD
return _balanceAt(state.balance, state.updateTime, state.maxEnd, receivers, timestamp); //@audit 3 SLOADS
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L539-L542

# [G-03] require()/revert() strings longer than 32 bytes cost extra gas

## Proof Of Concept
```solidity
require(dripsHash == 0, "Drips history entry with hash and receivers");
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L454

```solidity
require(timestamp >= state.updateTime, "Timestamp before last drips update");
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L540

```solidity
modifier onlyAdminOrPauser() {
    require(
        admin() == msg.sender || isPauser(msg.sender), "Caller is not the admin or a pauser"
    );
    _;
}
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L52-L57

```solidity
modifier onlyHolder(uint256 tokenId) {
    require(
        _isApprovedOrOwner(_msgSender(), tokenId),
        "ERC721: caller is not token owner or approved"
    );
    _;
}
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L40-L46

```solidity
require(prevUserId < userId, "Splits receivers not sorted by user ID");
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L239

```solidity
require(
    _hashSplits(currReceivers) == _splitsHash(userId), "Invalid current splits receivers"
);
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L254-L256

