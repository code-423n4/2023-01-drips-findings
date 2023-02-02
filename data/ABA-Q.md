# [01] Redundant duplication check on receivers in `_assertSplitsValid`

Context: https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L238-L239

In the `_assertSplitsValid` function from `Splits.sol`, the receiver list is checked for duplication and for being sorted.
The check for duplication `prevUserId != userId` is redundant (and wastes gas) as the next require conditions: `prevUserId < userId` also eliminates duplicates 

```Solidity
    if (i > 0) {
        require(prevUserId != userId, "Duplicate splits receivers");
        require(prevUserId < userId, "Splits receivers not sorted by user ID");
    }
```

## Recommended Mitigation Steps

Remove the first require: `require(prevUserId != userId, "Duplicate splits receivers");`


# [02] Missing, incorrect, or incomplete NatSpec comments
Within the codebase, many functions have missing, incorrect, or incomplete NatSpec comments.

- function `_receiveDripsResult` in `Drips.sol` ([context](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L269))
    - in the description of `@return amtPerCycle` _fromCycle_ should be used instead of _toCycle_

- function `_dripsRangeInFuture` in `Drips.sol` ([context](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L967-L974))
    - missing _updateTime_, _maxEnd_ parameters
    - missing _@return_ information

- function `_dripsRange` in `Drips.sol` ([context](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L978-L991))
    - missing _@return_ information

- function `_isOrdered` in `Drips.sol` ([context](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1058-L1065))
    - the last parameter _prev_ is actually _next_
    - missing _@return_ information

- function `_drippedAmt` in `Drips.sol` ([context](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1072-L1097))
    - the last natspec _amt_ should be _@return_ not _@param_
