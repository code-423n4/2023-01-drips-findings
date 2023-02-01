Instead of i++, use unchecked{++i} (or utilize assembly if possible).

Use "++i" instead of "i++". This optimization is particularly useful in "for" loops, but can be applied anywhere in your code. To save even more gas, use "unchecked{++i;}", but be aware that this does not check for overflow. If you're concerned about this, you can add a "require" statement after the loop to verify that i is equal to its final incremented value. For maximum gas savings, inline assembly can be used, although it has limitations. For instance, you cannot use Solidity syntax to make internal contract calls within an assembly block, and external calls must be done with "call()" or "delegatecall()" instruction. Nevertheless, when feasible, inline assembly can greatly reduce gas usage.

Code:

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L174
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L358
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L428
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L655
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L664
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L745
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L777
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L959
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L962
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L136
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L59
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L93
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231

