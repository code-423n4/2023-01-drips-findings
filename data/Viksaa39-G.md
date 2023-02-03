<x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables

There are 4 instances of this issue:

- [https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L636](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L636)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L99](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L99)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L168](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L168)

Function guaranteed to revert when called by unauthorised users can be marked payable 

These are 3 instances of this issue:

- [https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L386](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L386)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L109](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L109)

**Setting the Constructor to Payable**

We can save gas if we mark constructor as payable. 

There are 7 instances of this issue:

- [https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L73](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L73)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94)

Usage of uints/ints smaller than 32 bytes (256 bits) is less gas efficient 

Because of how the EVM works when using variables less than 32 bytes the EVM uses more gas to later convert them to 32 bytes.

There are 2 instances of this issue:

- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L284](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L284)
- [https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283)