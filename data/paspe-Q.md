#### [NC-01] 
## Deleting a mapping slot can cause gaps
#### Summary
Mapping should be deleted with replaced with the last element.
#### Github permalinks
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L248 

## Tools Used

Manual Review

## Recommended Mitigation Steps

Get the length of the mapping and take the last element and replace it.