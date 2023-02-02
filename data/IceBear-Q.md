## [N-01] LOCK PRAGMAS TO SPECIFIC COMPILER VERSION
[Drips.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L2)

[DripsHub.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L2)

[AddressDriver.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L2)

[Caller.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L2)

[ImmutableSplitsDriver.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L2)

[Managed.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L2)

[NFTDriver.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L2)


### Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103
### Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
solidity-specific/locking-pragmas

## [N-02] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
[Caller.sol#L207](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L207)
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).

## [S-01] GENERATE PERFECT CODE HEADERS EVERY TIME

### Description
I recommend using header for Solidity code layout and readability

https://github.com/transmissions11/headers