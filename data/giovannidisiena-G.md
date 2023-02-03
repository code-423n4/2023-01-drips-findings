#### **[G-01] Remove Unnecessary Stack Variable**
##### **Context:**
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L174

##### **Description:**
Given `currNonce` is the result of a post-increment and used only once, in the computation of `executeHash`, this stack variable is unnecessary.

##### **Recommendation**:
Remove the unnecessary `currNonce` variable and increment nonce directly in the computation of `executeHash`.

#### **[G-02] Simplify Loop Counter**
##### **Context:**
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L358-L364

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L422-L433

##### **Description:**
`squeezedRevIdxs` is indexed from oldest configuration to newest by `squeezedNum - i - 1` but this can be simplified, saving gas.

##### **Recommendation**:
Begin the loop counter from `uint256 i = 1;` as in `_squeezeDripsResult` and index with `squeezedNum - i` to avoid subtracting 1 in every loop iteration.