# keccak256 empty array issue
in [Splits.sol#L275](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L275), when __receivers__ is an empty array, ___hashSplits__ will return __bytes32(0)__.
At the same time, in [Splits.sol#L262](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L262), when __userId__ hasn't been stored in _splitsStorage, function ___splitsHash__ will return zero.
Combine two condition above, which will bypass function ___assertCurrSplits__ [Splits.sol#L250](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L250)
Which might cause issue
