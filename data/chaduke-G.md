G1. These checks can be eliminated to save gas since they will be checked again in later part of the execution. 
a) https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L622


G2. The ``_drippedAmt()`` calculates amount in terms of both ``cycleSecs`` and ``amtPerSec``. It first calculates the ``endedCycles`` and then make adjustment on amount on both ends.  We can save gas by calculating directly on seconds alone; see below.

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


G3. The ``_calcMaxEnd`` function calculates the maximum end time of drips based on two given hints and a binary search. This function can be improved to calculate a new function called ``(maxspend, maxend) = maxSpendMaxEnd()`` that returns the maximum drips that can be spent ```maxspend`` and the the maximum end time to spend the ``maxspend``. 
```script
function maxSpendMaxEnd(
        uint256 balance,
        uint256[] memory configs,
        uint256 configsLen,
    ) private view returns () {uint256 maxspend, uint256 maxend)
        unchecked {
            for (uint256 i = 0; i < configsLen; i++) {
                (uint256 amtPerSec, uint256 start, uint256 end) = _getConfig(configs, i);
             // slither-disable-next-line timestamp
                if (end > maxend) {
                    maxend = end;
                }
                maxspent += _drippedAmt(amtPerSec, start, end);
            }
       }
    }
```
Then we can use ``maxspend`` and ``maxend`` to interpolate the first hint ``hint0`` before using ``hint1`` and ``hint2``. The idea is that the estimate ``hint0`` will be accurate and speed up the search.
```diff
function _calcMaxEnd(
        uint128 balance,
        DripsReceiver[] memory receivers,
        uint32 hint1,
        uint32 hint2
    ) private view returns (uint32 maxEnd) {
        unchecked {
            (uint256[] memory configs, uint256 configsLen) = _buildConfigs(receivers);

            uint256 enoughEnd = _currTimestamp();
            // slither-disable-start incorrect-equality,timestamp
            if (configsLen == 0 || balance == 0) {
                return uint32(enoughEnd);
            }

            uint256 notEnoughEnd = type(uint32).max;
            if (_isBalanceEnough(balance, configs, configsLen, notEnoughEnd)) {
                return uint32(notEnoughEnd);
            }

+         (uint256 maxspend, uint256 maxend) = maxSpendMaxEnd(balance, configs, configslen);
+         uint32 hint0 = maxend * balance/maxspend; 
           
+         if(hint0 > enoughEnd && hint0 < notEnoughEnd){
+                 if (_isBalanceEnough(balance, configs, configsLen, hint1)) enoughEnd = hint0;
+                 else notEnoughEnd = hint0;   
+         }
         
            if (hint1 > enoughEnd && hint1 < notEnoughEnd) {
                if (_isBalanceEnough(balance, configs, configsLen, hint1)) {
                    enoughEnd = hint1;
                } else {
                    notEnoughEnd = hint1;
                }
            }

            if (hint2 > enoughEnd && hint2 < notEnoughEnd) {
                if (_isBalanceEnough(balance, configs, configsLen, hint2)) {
                    enoughEnd = hint2;
                } else {
                    notEnoughEnd = hint2;
                }
            }

            while (true) {
                uint256 end = (enoughEnd + notEnoughEnd) / 2;
                if (end == enoughEnd) {
                    return uint32(end);
                }
                if (_isBalanceEnough(balance, configs, configsLen, end)) {
                    enoughEnd = end;
                } else {
                    notEnoughEnd = end;
                }
            }
            // slither-disable-end incorrect-equality,timestamp
        }
    }

```

G4. The second requirement already implies the first requirement, therefore, the first requirement statement can be eliminated to save gas.
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L238-L239


G5. No need to execute this for-loop when ``receivedAmt ==  0``.
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L247-L249

We can save gas by checking whether ``receivedAmt !=  0``:
```javascript
if(receivedAmt != 0){
            for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
                delete amtDeltas[cycle];
            }
}
```
G6. We can cache the result of function call ``_currCycleStart()`` to save gas; it is used three times below.


[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L410-L426](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L410-L426)
```
       DripsState storage sender = _dripsStorage().states[assetId][senderId];
            historyHashes = _verifyDripsHistory(historyHash, dripsHistory, sender.dripsHistoryHash);
            // If the last update was not in the current cycle,
            // there's only the single latest history entry to squeeze in the current cycle.
            currCycleConfigs = 1;
            // slither-disable-next-line timestamp
            if (sender.updateTime >= _currCycleStart()) currCycleConfigs = sender.currCycleConfigs;
        }
        squeezedRevIdxs = new uint256[](dripsHistory.length);
        uint32[2 ** 32] storage nextSqueezed =
            _dripsStorage().states[assetId][userId].nextSqueezed[senderId];
        uint32 squeezeEndCap = _currTimestamp();
        for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
            DripsHistory memory drips = dripsHistory[dripsHistory.length - i];
            if (drips.receivers.length != 0) {
                uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
                if (squeezeStartCap < _currCycleStart()) squeezeStartCap = _currCycleStart();
```


