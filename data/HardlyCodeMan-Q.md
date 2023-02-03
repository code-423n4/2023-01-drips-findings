## 2023-01-drips Quality Assurance Report

- [Solidity 0.8.17 is very new and not suffieciently tested](#use-older-version)
- [Unspecific Compiler Version Pragma - pragma ^](#unspecified-pragma)

---

### Solidity 0.8.17 is very new and not suffieciently tested {#use-older-version}

[Solidity Changelog](https://github.com/ethereum/solidity/blob/develop/Changelog.md)
- 0.8.18 (2023-02-01)
- **0.8.17 (2022-09-08)**
- 0.8.16 (2022-08-08)
- ...
- 0.8.10 (2021-11-09)

Unexpected bugs can be reported in recent versions:
- Risks related to recent releases
- Risks of complex code generation changes
- Risks of new language features
- Risks of known bugs

Recommend use of non-legacy, more time hardened version 0.8.10

### 8 Affected contracts
- [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L2)
- [Caller.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L2)
- [Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L2)
- [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L2)
- [ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L2)
- [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L2)
- [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L2)
- [Splits.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L2)
---

### Unspecific Compiler Version Pragma - pragma ^ {#unspecified-pragma}

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain. Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
[SWC-103](https://swcregistry.io/docs/SWC-103)

Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
solidity-specific/locking-pragmas

### 8 Affected contracts
- [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L2)
- [Caller.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L2)
- [Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L2)
- [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L2)
- [ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L2)
- [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L2)
- [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L2)
- [Splits.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L2)