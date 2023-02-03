# Gas Optimizations Report for Drips Protocol contest
## Overview
During the audit, 2 gas issues were found.  
Total savings ~1050.

№ | Title | Instance Count | Saved
--- | --- | --- | ---
G-1 | Use unchecked blocks for incrementing | 23 | 805
G-2 | Use unchecked blocks for subtractions where underflow is impossible | 7 | 245

## Gas Optimizations Findings(2)
### G-1. Use unchecked blocks for incrementing
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. In the loops or similar cases, "i" will not overflow because the loop will run out of gas before that or max value never be reached.

##### Instances
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L358
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L655
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L664
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L745
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L777
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L959
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L962
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L136
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231
- https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L93
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L174
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196
- https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L59
- https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61

##### Recommendation
Change:
```
for (uint256 i; i < n; ++i) {
 // ...
}
```
to:
```
for (uint256 i; i < n;) { 
 // ...
 unchecked { ++i; }
}
```

##### Saved
This saves ~30-40 gas per iteration.  
So, ~35*23 = 805
#
### G-2. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283 (2 subtractions as 2 instances)
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L361  (2 subtractions as 2 instances)
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L423
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L425
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L780

##### Saved
This saves ~35.  
So, ~35*7 = 245