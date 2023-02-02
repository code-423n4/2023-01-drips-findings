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


# 3: USE OF BLOCK.TIMESTAMP

Vulnerability details

## Context:

Block timestamps have been used historically for a number of purposes, including entropy for random numbers (see the Entropy Illusion for more information), locking funds for a set amount of time, and numerous state-changing time-dependent conditional statements. The ability of miners to slightly modify timestamps can be risky if block timestamps are used improperly in smart contracts.

Reference: https://swcregistry.io/docs/SWC-116 

## Proof of Concept

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1129 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L173 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

1. Block timestamps shouldn't be utilized to create random numbers or entropy.

2. Use of trusted oracles


# 4: PRAGMA VERSION ^0.8.17 VERSION TOO RECENT TO BE TRUSTED

Vulnerability details

### Context:

> #### Unexpected bugs can be reported in recent versions including:

> - Risks related to recent releases
> - Risks of complex code generation changes
> - Risks of new language features
> - Risks of known bugs

For reference, see https://github.com/ethereum/solidity/blob/develop/Changelog.md

0.8.18 (2023-02-01)
0.8.17 (2022-09-08)
0.8.16 (2022-08-08)
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)

## Proof of Concept

All contracts in scope

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Use a non-legacy and more battle-tested version consider using 0.8.10

# 5: USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()

###Context:

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

## Proof of Concept

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L207 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Rather than using abi.encodePacked for appending bytes, use bytes.concat() 


# 6: GENERATE PERFECT CODE HEADERS EVERY TIME

Vulnerability details

### Context:

It is recommended to use a header for Solidity code layout and readability.

For reference, see https://github.com/transmissions11/headers

## Proof of Concept

All contracts in scope

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/


