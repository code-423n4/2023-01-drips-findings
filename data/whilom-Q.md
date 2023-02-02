
## [I-01] Unecessary `finalAmtPerCycle` Check 

Line: https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L252

There is no need to check whether `finalAmtPerCycle != 0` since adding by `0` results in the same number.  
```
if (finalAmtPerCycle != 0) {
    amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
}
```

## [G-01] Early break in `_calcMaxEnd()` when `balance == 0`
Line: https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L691

We can check `balance == 0` and immediately return `uint32(enoughEnd)` prior to calling `_buildConfigs` which will save gas. This is true especially in case where receivers array is long but `balance == 0` anyway. 

## [L-01] `_isBalanceEnough` returns `false` when `start > end` 
Line: https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L737

Consider the case where we call isBalanceEnough with 
```
_isBalanceEnough
balance: 100
configs: [18446744073709551701899345930]
configsLen: 1
maxEnd: 25

Where the passed config number is 
uint256: amtPerSec 1000000000
uint256: start 20
uint256: end 10
```
- This should return `true` since `start > end`. However since there is no check and `_drippedAmt(amtPerSec, start, end)` underflows to a large value, then `spent > balance` and the function returns false. 
- To fix, add a check in `_isBalanceEnough`

```
From: 
if (maxEnd <= start) {
  continue;
}
To: 
if (maxEnd <= start || start >= end) {
  continue;
}
```

Note: this is not an issue in other uses of the `_drippedAmt` since `_dripsRange` is used to get `start, end` which guarantees that `start <= end`.