# Missing Natspec for functions
## Instances

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L629

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L170 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L635

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L202

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L98

## Recommendation
Add natspec for these functions and their parameters so it will more user readable. 

# Changing Admin can be two step process

## Description
Admin change is a crucial process. Admin has more authority in the protocol like pausing, adding pausers, revoking pausers etc.
 
## Instance
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84

## Recommendation
This process can be 2 step process: request and accept transactions, through this more control over the admin change.  

