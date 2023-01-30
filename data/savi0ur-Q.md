# [L-01] Incorrect comparision of drips config

Its expected to first compares their `amtPerSec`s, then their `start`s and then their `duration`s. But since during config creation, it stores `dripId` in higher order bits of config which will make comparision first on `dripId`. It might give wrong result while checking `_isOrdered` function.

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L94-L96

# [L-02] Incorrect calculation of cycle containing the given timestamp

Its adding +1 blindly while calculating cycle containing the given timestamp. It will give wrong cycle at edges. Say we have `cycleSecs = 5` and `timestamp = 10`, it will output cycle containing timestamp as `timestamp / cycleSecs + 1 = 10 / 5 + 1 = 3` instead of correct cycle `2`, as cycle#1: 1 to 5, cycle#2: 6 to 10, cycle#3: 11 to 15.

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1120-L1124

It should calculate it as `timestamp / cycleSecs + ((timestamp % cycleSecs) > 0)`

