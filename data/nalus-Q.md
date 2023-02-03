# 1 Comment is wrong
First, dripId_ is compared.

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L93

# 2 _balanceAt function name shadowing

Change one of the function's name to avoid confusion

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L533
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L555

# 3 Store magic numbers in constants

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L41

# 4 NFTDriver.sol is missing tokenUri function to return erc721 metadata.

# 5 DripsHub says cyclesSecs must be above 1, but there is no condition checking it

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L109-L114

# 6 finalAmtPerCycle sounds like average naming it finalCycleAmt would be more appropriate

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L240-L254

# 7 Protocol design is not optimal for funds withdrawal

Add a withdrawable function that allows the withdrawal of splittable funds so that the user doesn't need to call 3 functions (_setSplits, _split, collect) in order to withdraw funds that one receives from a split.
