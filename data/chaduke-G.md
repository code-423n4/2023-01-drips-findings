G1. The ``_drippedAmt()`` calculates amount in terms of both ``cycleSecs`` and ``amtPerSec``. It first calculates the ``endedCycles`` and then make adjustment on amount on both ends.  We can save gas by calculating directly on seconds alone; see below.

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1093-L1115

```
function _drippedAmt(uint256 amtPerSec, uint256 start, uint256 end)
        private
        view
        returns (uint256 amt)
    {
       assembly {
            let elapsedSecs := sub(end, start)
            amt := div(mul(elsapsedSecs, amtPerSec), _AMT_PER_SEC_MULTIPLIER)
        }
    }
```