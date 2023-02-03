Gas optimisation at
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L418
dripsHistory.length can be saved in a local variable to save gas on calls at lines 418, 422, 423

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L449
dripsHistory.length dripsHistory.length can be saved in a local variable to save gas on calls at lines 450, 451