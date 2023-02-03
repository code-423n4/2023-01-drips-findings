## [QA Report]

### [NC-1] Unused Imports are used
There are 3 instances of it and there permalinks are
[AddressDriver.sol#L5](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L5), 
[NFTDriver.sol#L4](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L4) (here only `DripsHistory` was not used) & [DripsHub.sol#L4](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L4) (here only `DripsConfigImpl` was not used)
it is recommend to remove unused imports for better readability of the code

### [NC-2] Critical address changes should use two step procedure
There is one instance of it and there its recommend to use two step procedure for changing admin address
 https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84

### [NC-3] LOCK PRAGMAS TO SPECIFIC COMPILER VERSION
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
Ref- https://swcregistry.io/docs/SWC-103
context: All contracts (its recommend to  Lock pragmas to specific compiler version)

### [NC-4] USE OF `BYTES.CONCAT()` INSTEAD OF `ABI.ENCODEPACKED()`
There are 5 instances of it  and there permalinks
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L842,
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L858,
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L278,
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L176,
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L207
 its recommend to use two step procedure while changing the admin address
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled.
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

