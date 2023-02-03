
# Low

| | Issue index |
| ----------- | ----------- |
| 1 | [Missing checks for address(0x0) when assigning values to `address` state or `immutable` variables](#missing-checks-for-address(0x0)-when-assigning-values-to-`address`-state-or-`immutable`-variables) |
| 2 | [`block.timestamp` used as time proxy](#`block.timestamp`-used-as-time-proxy) |
| 3 | [Destination recipient for assets may be `address(0)`](#destination-recipient-for-assets-may-be-`address(0)`) |
| 4 | [Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions](#upgradeable-contract-is-missing-a-`__gap[50]`-storage-variable-to-allow-for-new-storage-variables-in-later-versions) |
| 5 | [`_safeMint()` should be used rather than `_mint()` wherever possible](#`_safemint()`-should-be-used-rather-than-`_mint()`-wherever-possible) |

## Missing checks for address(0x0) when assigning values to `address` state or `immutable` variables

`NOTE`: None of these findings where found by [4naly3er output - NC](LINKKKKKKK)

### Summary

Zero address should be checked for state variables, immutable variables. A zero address can lead into problems.

### Code Snippet

- [AddressDriver.sol#L31](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/AddressDriver.sol#L31

- [ImmutableSplitsDriver.sol#L29](https://github.com/code-423n4/2023-01-drips/blob/1e5917b4e61b6a52a296a857ebbc523b22a95689/src/ImmutableSplitsDriver.sol#L29

- [Managed.sol#L158](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/Managed.sol#L158

- [NFTDriver.sol#L33](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/NFTDriver.sol#L33)

- [NFTDriver.sol#L36](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/NFTDriver.sol#L36)

Some constructor uses some OZ constructors, at them there is no check as can be seen at:

- [ERC2771Context.sol#L17](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/metatx/ERC2771Context.sol#L17)

- [ERC1967Upgrade.sol#L123](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/ERC1967/ERC1967Upgrade.sol#L123

### Recommendation

Check zero address before assigning or using it

## `block.timestamp` used as time proxy 
### Summary

Risk of using `block.timestamp` for time should be considered. 

### Details

`block.timestamp` is not an ideal proxy for time because of issues with synchronization, miner manipulation and changing block times. 

This kind of issue may affect the code allowing or reverting the code before the expected deadline, modifying the normal functioning or reverting sometimes.

### References

SWC ID: 116

### Code Snippet 

- [Drips.sol#L1129](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1129)
        return uint32(block.timestamp);

- [Caller.sol#L173](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L173)
        require(block.timestamp <= deadline, "Execution deadline expired");

### Recommendation

- Consider the risk of using `block.timestamp` as time proxy and evaluate if block numbers can be used as an approximation for the application logic. Both have risks that need to be factored in. 
- Consider using an oracle for precision



## Destination recipient for assets may be `address(0)`

- [AddressDriver.sol#L62](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/AddressDriver.sol#L62)

- [AddressDriver.sol#L145](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/AddressDriver.sol#L145)

- [NFTDriver.sol#L116](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/NFTDriver.sol#L116)

- [NFTDriver.sol#L204](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/NFTDriver.sol#L204)

### Description

The recipient of a transfer may be `address(0)`, leading to lost assets.


## Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
### Impact 

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments" (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely. [See](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps).

For a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

### Vulnerability Details

In the following context of the upgradeable contracts they are expected to use gaps for avoiding collision:

- `UUPS`

However, none of these contracts contain storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Refer to the bottom part of this [article](https://docs.openzeppelin.com/contracts/4.x/upgradeable).

If a contract inheriting from a base contract contains additional variable, then the base contract cannot be upgraded to include any additional variable, because it would overwrite the variable declared in its child contract. This greatly limits contract upgradeability.

### Code Snippet

- [Managed.sol#L18](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/Managed.sol#L18)

### Tools Used

Manual analysis

### Recommendation

Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. 

`uint256[50] private __gap;`

Reference OpenZeppelin [upgradeable contract templates](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable).

## `_safeMint()` should be used rather than `_mint()` wherever possible
### Impact 

In `NFTDriver.sol`, eventually it is called ERC721 `_mint()`. Calling `_mint()` this way does not ensure that the receiver of the NFT is able to accept them, making possible to lose them. 

`_safeMint()` should be used with as it checks to see if a user can properly accept an NFT and reverts otherwise.

There is no check of the address provided by the mint NFT that it implements `ERC721Receiver`. 

##### Details 

`_mint()` is discouraged in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. 

Both open OpenZeppelin and solmate have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

[References](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271)

### Code Snippet

- [NFTDriver.sol#L68](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L68)
        _mint(to, tokenId);

### Recommendation

Use `_safeMint()` as suggested by OpenZeppelin or include the check before minting.




# Informational

| | Issue index |
| ----------- | ----------- |
| 1 | [Use of deprecated draft file](#use-of-deprecated-draft-file) |
| 2 | [Constants should be defined rather than using magic numbers](#constants-should-be-defined-rather-than-using-magic-numbers) |
| 3 | [Max value can be used rather than `2 ** n`](#max-value-can-be-used-rather-than-`2-**-n`) |
| 4 | [Omissions in events](#omissions-in-events) |
| 5 | [Missing initializer modifier on constructor](#missing-initializer-modifier-on-constructor) |
| 6 | [No need to return void in a constructor](#no-need-to-return-void-in-a-constructor) |

## Use of deprecated draft file

There is a released file that is no longer a draft, it should be used rather than draft files

- [Caller.sol#L5](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L5)
import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";

## Constants should be defined rather than using magic numbers

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/supernovahs/ed0ecfe251e3adf824bb669be981bca9)


- [Drips.sol#L66](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L66)
        config = (config << 160) | amtPerSec_;

- [Drips.sol#L67](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L67)
        config = (config << 32) | start_;

- [Drips.sol#L68](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L68)
        config = (config << 32) | duration_;

- [Drips.sol#L74](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L74)
        return uint32(DripsConfig.unwrap(config) >> 224);

- [Drips.sol#L79](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L79)
        return uint160(DripsConfig.unwrap(config) >> 64);

- [Drips.sol#L84](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L84)
        return uint32(DripsConfig.unwrap(config) >> 32);

- [Drips.sol#L185](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L185)
        mapping(uint256 => uint32[2 ** 32]) nextSqueezed;

- [Drips.sol#L357](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L357)
        uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];

- [Drips.sol#L419](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L419)
        uint32[2 ** 32] storage nextSqueezed =

- [Drips.sol#L805](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L805)
        configs[configsLen] = (amtPerSec << 64) | (start << 32) | end;

- [Drips.sol#L825](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L825)
        return (val >> 64, uint32(val >> 32), uint32(val));

- [Drips.sol#L805](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L805)
        configs[configsLen] = (amtPerSec << 64) | (start << 32) | end;

- [Drips.sol#L825](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L825)
        return (val >> 64, uint32(val >> 32), uint32(val));

- [NFTDriver.sol#L51](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L51)
        return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_mintedTokensSlot).value;

- [AddressDriver.sol#L41](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L41)
        return (uint256(driverId) << 224) | uint160(userAddr);

- [ImmutableSplitsDriver.sol#L37](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L37)
        return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_counterSlot).value;


## Max value can be used rather than `2 ** n`

Rather than using 2 ** 256 or other numbers (128, 64, 32, 16, 8...), it can be used type(uint256).max (depending on the exponentiation number).

- [Drips.sol#L185](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L185)
        mapping(uint256 => uint32[2 ** 32]) nextSqueezed;

- [Drips.sol#L357](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L357)
        uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];

- [Drips.sol#L419](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L419)
        uint32[2 ** 32] storage nextSqueezed =

## Omissions in events

The events can include the new value and old value and even be refactored in just one event
 
`_managedStorage().isPaused`

- [Managed.sol#L124](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L124)
        emit Paused(msg.sender);

- [Managed.sol#L130](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L130)
        emit Unpaused(msg.sender);

## Missing initializer modifier on constructor

OpenZeppelin [recommends](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5) that the initializer modifier be applied to constructors in order to avoid potential griefs, [social engineering](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/4), or exploits. 

- [Managed.sol#L73](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/Managed.sol#L73)

## No need to return void in a constructor

- [DripsHub.sol#L113](https://github.com/code-423n4/2023-01-drips/blob/a271fcc95ed1f2953bfef2345c86c0035a1dffbf/src/DripsHub.sol#L113)
