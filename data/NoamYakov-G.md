
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 22 |  1170 |
| [G&#x2011;02] | Use custom errors rather than `revert()`/`require()` strings to save gas | 3 |  150 |
| [G&#x2011;03] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too) | 6 |  678 |

Total: 31 instances over 3 issues with **1998 gas** saved.

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions.

## Gas Optimizations

### [G&#x2011;01]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

*There are 22 instances of this issue:*

```solidity
File: src\Caller.sol

174         uint256 currNonce = nonce[sender]++;

196         for (uint256 i = 0; i < calls.length; i++) {
```

```solidity
File: src\DripsHub.sol

247             for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

287         for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

358         for (uint256 i = 0; i < squeezedNum; i++) {

422         for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

/// @audit inside `for`-loop
428                     squeezedRevIdxs[squeezedNum++] = i;

450         for (uint256 i = 0; i < dripsHistory.length; i++) {

490         for (; idx < receivers.length; idx++) {

563         for (uint256 i = 0; i < receivers.length; i++) {

655             state.currCycleConfigs++;

664             for (uint256 i = 0; i < newReceivers.length; i++) {

/// @audit inside `while`-loop
959                 currIdx++;

/// @audit inside `while`-loop
962                 newIdx++;
```

```solidity
File: src\DripsHub.sol

136         driverId = dripsHubStorage.nextDriverId++;

613         for (uint256 i = 0; i < userMetadata.length; i++) {
```

```solidity
File: src\ImmutableSplitsDriver.sol

59          StorageSlot.getUint256Slot(_counterSlot).value++;

61          for (uint256 i = 0; i < receivers.length; i++) {
```

```solidity
File: src\NFTDriver.sol

93          StorageSlot.getUint256Slot(_mintedTokensSlot).value++;
```

```solidity
File: src\Splits.sol

127         for (uint256 i = 0; i < currReceivers.length; i++) {

158         for (uint256 i = 0; i < currReceivers.length; i++) {

231         for (uint256 i = 0; i < receivers.length; i++) {
```

### [G&#x2011;02]  Use custom errors rather than `revert()`/`require()` strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas.

*There are 3 instances of this issue:*

```solidity
File: src\Managed.sol

53          require(
54              admin() == msg.sender || isPauser(msg.sender), "Caller is not the admin or a pauser"
55          );
```

```solidity
File: src\NFTDriver.sol

41          require(
42              _isApprovedOrOwner(_msgSender(), tokenId),
43              "ERC721: caller is not token owner or approved"
44        );
```

```solidity
File: src\Splits.sol

254         require(
255             _hashSplits(currReceivers) == _splitsHash(userId), "Invalid current splits receivers"
256         );
```

### [G&#x2011;03]  `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too)
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**. Subtructions act the same way.

*There are 6 instances of this issue:*

```solidity
File: src\Drips.sol

253                     amtDeltas[toCycle].thisCycle += finalAmtPerCycle;

1053                amtDelta.thisCycle += int128(fullCycle - nextCycle);

1054                amtDelta.nextCycle += int128(nextCycle);
```

```solidity
File: src\DripsHub.sol

632         totalBalances[erc20] += amt;
```

```solidity
File: src\Splits.sol

99          _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;

168         balance.collectable += collectableAmt;
```
