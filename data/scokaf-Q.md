# 1: FLOATING PRAGMA 

Vulnerability details

### Impact

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.17, ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

For reference, see https://swcregistry.io/docs/SWC-103

## Proof of Concept

All contracts in scope 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Remove ^ in “pragma solidity ^0.8.17” and change it to “pragma solidity 0.8.17” to be consistent with the rest of the contracts.

# 2: USE CONSTANTS FOR NUMBERS

Vulnerability details

### Context:

In several locations in the code, values like: type(uint32).max are used. It is quite easy to make a mistake somewhere, also when comparing values.

## Proof of Concept

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L695 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L975 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L289 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L174 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Recommend defining constants for the numbers and values used throughout the code.

