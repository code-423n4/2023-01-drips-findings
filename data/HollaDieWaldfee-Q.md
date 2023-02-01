# Drips Protocol Low Risk and Non-Critical Issues
## Summary
| Risk      | Title | File | Instances
| ----------- | ----------- | ----------- | ----------- |
| L-01      | Unlocked pragma | - | 8 |
| L-02      | Use 2-Step-Process to change admin | Managed.sol | 1 |
| L-03      | Check that driver address is not zero address | DripsHub.sol | 2 |
| L-04      | Application will be unusable by the year 2106 | Drips.sol | 1 |
| L-05      | `name` and `symbol` of `NFTDriver` cannot be accessed via Proxy | NFTDriver.sol | 1 |
| N-01      | Remove unnecessary imports | - | 3 |
| N-02      | Remove unnecessary `_MAX_TOTAL_SPLITS_BALANCE` variable | Splits.sol | 1 |
| N-03      | `require` statement is redundant | Splits.sol | 1 |
| N-04      | `DripsReceiverSeen` event should include `assetId` | Drips.sol | 1 |


## [L-01] Unlocked pragma
Currently the Solidity source files will compile with any version `>=0.8.17` and `<0.9.0`.  
It is best practice to use a fixed compiler version.  

There are 8 instances of this issue:  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L2)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L2)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L2)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L2)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L2)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L2)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L2)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L2](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L2)  

## [L-02] Use 2-Step-Process to change admin
It is best practice to implement a 2-Step-Process for changing the `admin` of a contract.  
The 2-Step-Process involves setting a `pending admin` first. The `pending admin` must than call a function to approve the change of the admin address.  

This prevents accidentally setting a wrong admin address which would result in a loss of admin access to the contract.  

This 2-Step-Process should be implemented in the `Managed.sol` contract ([https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L18](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L18)).  

## [L-03] Check that driver address is not zero address
The `DripsHub.registerDriver` and `DripsHub.updateDriverAddress` functions should check that the driver address that is registered is not the zero address.  

This is especially important for `DripsHub.updateDriverAddress`. Registering the zero address would mean that access to the `driverId` of this driver is lost.  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L134-L139](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L134-L139)  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152-L156](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152-L156)  

## [L-04] Application will be unusable by the year 2106
The `Drips` contract converts timestamps to `uint32` in the `_currTimestamp` function ([https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1128-L1130](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1128-L1130)).  

The `_currTimestamp` function is necessary for the operation of the `Drips` contract and thereby the whole protocol.  

Using `uint32` to hold timestamps means that the protocol becomes inoperable by the year ~2106.  

The issue with this is that it severly limits the potential of the Drips protocol.  
Many other protocols may chose not to integrate with Drips because of this limitation.  

A protocol that wants to make sure it can operate for as long as possible without a built-in "kill date" cannot use the Drips protocol.  

When the `type(uint32).max` timestamp is reached, funds are stuck in the `Drips` contract. `Drips` can no longer be received even when the time at which they were streamed was before this timestamp.  

## [L-05] `name` and `symbol` of `NFTDriver` cannot be accessed via Proxy
The OpenZeppelin `ERC721` contract which `NFTDriver` inherits stores `_name` and `_symbol` in storage ([https://github.com/OpenZeppelin/openzeppelin-contracts/blob/501a78e13426ec10249e44386239c74783478dd0/contracts/token/ERC721/ERC721.sol#L24-L27](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/501a78e13426ec10249e44386239c74783478dd0/contracts/token/ERC721/ERC721.sol#L24-L27)).  

Also, `_name` and `_symbol` are set in the constructor of the `ERC721` contract ([https://github.com/OpenZeppelin/openzeppelin-contracts/blob/501a78e13426ec10249e44386239c74783478dd0/contracts/token/ERC721/ERC721.sol#L44-L47](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/501a78e13426ec10249e44386239c74783478dd0/contracts/token/ERC721/ERC721.sol#L44-L47)).  

Thereby, when `name` and `symbol` are queried from the Proxy, empty bytes are returned.  

This is bad because the Proxy contract should obviously return the `name` and `symbol` of the `NFTDriver`. This can be solved by overriding the `name` and `symbol` functions in the `NFTDriver` contract to just return "DripsHub identity" and "DHI" without needing to read from any variable.  

## [N-01] Remove unnecessary imports
Solidity files should only import the dependencies that are necessary to make the code cleaner and make it easier to understand.  

There is 3 instance of this:  

`DripsHistory`  
[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L5](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L5)  

`StorageSlot`  
[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L6](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L6)  

`DripsConfig`  
[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L4](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L4)  

## [N-02] Remove unnecessary `_MAX_TOTAL_SPLITS_BALANCE` variable
The `_MAX_TOTAL_SPLITS_BALANCE` variable in `Splits.sol` is never used. So it can be removed.  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L25](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L25)  

## [N-03] `require` statement is redundant
The `Splits._assetSplitsValid` function contains two `require` statements that check the user IDs:  

[https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L238-L239](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L238-L239)  
```solidity
require(prevUserId != userId, "Duplicate splits receivers");
require(prevUserId < userId, "Splits receivers not sorted by user ID");
```

The first `require` statement is not necessary. If the first `require` statement fails then the second fails as well.  

There is a small benefit for having separate error messages for both cases.  

However because the first `require` statement is not necessary it should be removed which also saves Gas.  

## [N-04] `DripsReceiverSeen` event should include `assetId`
The `DripsReceiverSeen` event is emitted in the `_setDrips` function for each receiver ([https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L666](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L666)).  

Each asset has its own drips receivers.  

Therefore the event should also emit the `assetId` to describe which asset the receiver is set for.  
