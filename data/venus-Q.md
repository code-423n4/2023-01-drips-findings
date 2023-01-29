https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L157

splitsWeight variable's default value is already 0. Declaring its value to the same value, zero is redundant. Redundant declarations cause extra Gas consumption and decrease code readability.