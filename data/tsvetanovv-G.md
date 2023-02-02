### GAS Issues List
| Number |Issues Details
|:--:|:-------|:--:|
|[GAS-01]| Unchecking arithmetics operations that can’t underflow/overflow
|[GAS-02]| `x = x - y` costs less gas than `x -= y`, same for addition
|[GAS-03]| Using storage instead of memory for structs/arrays saves gas
|[GAS-04]|Before some functions, we should check some variables for possible gas save
***

## [GAS-01] UNCHECKING ARITHMETICS OPERATIONS THAT CAN’T UNDERFLOW/OVERFLOW
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.
[Reference](https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic)

```
Drips.sol
247: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
287: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
422: for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
450: for (uint256 i = 0; i < dripsHistory.length; i++) {
490: for (; idx < receivers.length; idx++) {
563: for (uint256 i = 0; i < receivers.length; i++) {
664: for (uint256 i = 0; i < newReceivers.length; i++) {

DripsHub.sol
613: for (uint256 i = 0; i < userMetadata.length; i++) {

Splits.sol
127: for (uint256 i = 0; i < currReceivers.length; i++) {
158: for (uint256 i = 0; i < currReceivers.length; i++) {
231: for (uint256 i = 0; i < receivers.length; i++) {

Caller.sol
196: for (uint256 i = 0; i < calls.length; i++) {

ImmutableSplitsDriver.sol
61: for (uint256 i = 0; i < receivers.length; i++) {
```
***

## [GAS-02] `x = x - y` costs less gas than `x -= y`, same for addition

```
Drips.sol
253: amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
288: amtPerCycle += state.amtDeltas[cycle].thisCycle;
289: receivedAmt += uint128(amtPerCycle);
290: amtPerCycle += state.amtDeltas[cycle].nextCycle;
429: amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);
495: amt += _drippedAmt(receiver.config.amtPerSec(), start, end);
572: balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
755: spent += _drippedAmt(amtPerSec, start, end);
777: for (uint256 i = 0; i < receivers.length; i++) {
1053: amtDelta.thisCycle += int128(fullCycle - nextCycle);
1054: amtDelta.nextCycle += int128(nextCycle);

DripsHub.sol
632: totalBalances[erc20] += amt
636:  _dripsHubStorage().totalBalances[erc20] -= amt;

Splits.sol
99: _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
159: splitsWeight += currReceivers[i].weight;
162: splitAmt += currSplitAmt;
167: collectableAmt -= splitAmt;
168: balance.collectable += collectableAmt;

ImmutableSplitsDriver.sol
62: weightSum += receivers[i].weight;
```

You can replace all `-=` and `+=` occurrences to save gas
***

## [GAS-03] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```
Drips.sol
350: uint256[] memory squeezedRevIdxs;
351: bytes32[] memory historyHashes;
355: bytes32[] memory squeezedHistoryHashes
358: for (uint256 i = 0; i < squeezedNum; i++) {
```
***

## [GAS-04] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:
```
DripsHub.sol
629: function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {
630:        mapping(IERC20 => uint256) storage totalBalances = _dripsHubStorage().totalBalances;
631:        require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");
632:        totalBalances[erc20] += amt;

635: function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {
636:        _dripsHubStorage().totalBalances[erc20] -= amt;
```