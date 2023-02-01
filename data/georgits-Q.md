## Use latest Solidity version with a stable pragma statement
[Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol), [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol), [Splits.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol), [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol), [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol), [Caller.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol), [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol), [ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol)

## Remove unused imports
Managed.sol - [6](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L6)

NFTDriver.sol - [4](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L4)(DripsHistory)

AddressDriver.sol - [5](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L5)

DripsHub.sol - [4](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L4)(DripsConfig, DripsConfigImpl)


## Missing zero address checks
Managed.sol - [158](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L158)

AddressDriver.sol - [30](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30)

NFTDriver.sol - [32](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32)

## Remove unnecessary `require` statement
This require statement is unnecessary
```
require(prevUserId != userId, "Duplicate splits receivers");
```
since the require statement on the next line covers it.
```
require(prevUserId < userId, "Splits receivers not sorted by user ID");
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L238-L239