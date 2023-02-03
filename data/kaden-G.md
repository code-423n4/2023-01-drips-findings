##### Redundant require statement
```js
require(prevUserId != userId, "Duplicate splits receivers");
require(prevUserId < userId, "Splits receivers not sorted by user ID");
```

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L238

##### Short circuit no receivable drips
In the case that there are no drips to be received in a call to receiveDrips, the function fully executes with no state changes, including an event emitted with an amount of 0. This logic is wasteful and redundant and instead it may make more sense to revert or at least immediately return 0 in `_receiveDrips` if a revert is unwanted.

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L238

##### Use external function visibility over public when relevant
Often functions are marked as public when they are not being called from within the contract. Explicitly labeling external functions forces the function parameter storage location to be set as calldata, which saves gas each time the function is executed.

##### Tokens mistakenly sent will be permanently locked
The drips contracts use internal acounting rather than checking the actual token balance of the contracts, as a result, if the token balance exceeds the internal accounting balance, all excess tokens are unrecoverable