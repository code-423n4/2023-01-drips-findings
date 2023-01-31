## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Add to `blacklist` function | 1 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Draft Import Dependencies | 1 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Possible rounding issue | 7 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Use `_safeMint` instead of `_mint` | 1 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Missing Checks for Address(0x0)  | 2 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Contracts are not using their OZ Upgradeable counterparts | 16 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Remove unused code | 2 |

Total: 30 contexts over 7 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Add a timelock to critical functions | 7 |
| [NC&#x2011;2](#NC&#x2011;2) | Avoid Floating Pragmas: The Version Should Be Locked | 8 |
| [NC&#x2011;3](#NC&#x2011;3) | Critical Changes Should Use Two-step Procedure | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Function writing that does not comply with the Solidity Style Guide  | 8 |
| [NC&#x2011;5](#NC&#x2011;5) | Use `delete` to Clear Variables | 2 |
| [NC&#x2011;6](#NC&#x2011;6) | Imports can be grouped together | 16 |
| [NC&#x2011;7](#NC&#x2011;7) | NatSpec return parameters should be included in contracts | 1 |
| [NC&#x2011;8](#NC&#x2011;8) | No need to initialize uints to zero | 9 |
| [NC&#x2011;9](#NC&#x2011;9) | Missing event for critical parameter change | 4 |
| [NC&#x2011;10](#NC&#x2011;10) | Implementation contract may not be initialized | 8 |
| [NC&#x2011;11](#NC&#x2011;11) | NatSpec comments should be increased in contracts | 1 |
| [NC&#x2011;12](#NC&#x2011;12) | Use `bytes.concat()` | 1 |
| [NC&#x2011;13](#NC&#x2011;13) | Use of Block.Timestamp | 1 |

Total: 67 contexts over 13 issues

## Low Risk Issues

### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.






### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Draft Import Dependencies
Contracts are importing draft dependencies. These imported contracts are still a draft and are not considered ready for mainnet use and have not received adequate security auditing or are liable to change with future development.

#### <ins>Proof Of Concept</ins>


```solidity
5: import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L5



#### <ins>Recommended Mitigation Steps</ins>

Stop using draft imported contracts and use their release version if it exists.



### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Possible rounding issue

There might be a rounding issue as it divides seperately.

#### <ins>Proof Of Concept</ins>


```solidity
1045: int256 fullCycle = (int256(uint256(_cycleSecs)) * amtPerSec) / amtPerSecMultiplier;
1047: int256 nextCycle = (int256(timestamp % _cycleSecs) * amtPerSec) / amtPerSecMultiplier;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1045

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1047



```solidity
1122: return timestamp / _cycleSecs + 1;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1122

```solidity
130: splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L130

```solidity
161: uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L161






### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Use `_safeMint` instead of `_mint`

According to openzepplin's ERC721, the use of `_mint` is discouraged, use _safeMint whenever possible.
https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_mint-address-uint256-

#### <ins>Proof Of Concept</ins>


```solidity
68: _mint(to, tokenId);
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L68



#### <ins>Recommended Mitigation Steps</ins>

Use `_safeMint` whenever possible instead of `_mint`






### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Missing Checks for Address(0x0) 

Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

#### <ins>Proof Of Concept</ins>


```solidity
134: function registerDriver: address driverAddr
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L134

```solidity
152: function updateDriverAddress: address newDriverAddr
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L152



#### <ins>Recommended Mitigation Steps</ins>

Consider adding explicit zero-address validation on input parameters of address type.



### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
13: import {ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
14: import {SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L13-L14

```solidity
4: import {Address} from "openzeppelin-contracts/utils/Address.sol";
5: import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";
6: import {ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
7: import {EnumerableSet} from "openzeppelin-contracts/utils/structs/EnumerableSet.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L4-L7

```solidity
7: import {IERC20} from "openzeppelin-contracts/token/ERC20/IERC20.sol";
8: import {SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L7-L8

```solidity
6: import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L6

```solidity
4: import {UUPSUpgradeable} from "openzeppelin-contracts/proxy/utils/UUPSUpgradeable.sol";
5: import {ERC1967Proxy} from "openzeppelin-contracts/proxy/ERC1967/ERC1967Proxy.sol";
6: import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";
7: import {EnumerableSet} from "openzeppelin-contracts/utils/structs/EnumerableSet.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L4-L7

```solidity
6: import {Context, ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
7: import {IERC20, SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";
8: import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L6-L8



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts



### <a href="#Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Remove unused code
This code is not used in the main project contract files, remove it or add event-emit. It is only used in test files.
Code that is not in use, suggests that they should not be present and could potentially contain insecure functionalities.

#### <ins>Proof Of Concept</ins>


```solidity
function create
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L60







## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

#### <ins>Proof Of Concept</ins>


```solidity
122: function setDrips(
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L122

```solidity
158: function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L158

```solidity
510: function setDrips(
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L510

```solidity
576: function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L576

```solidity
84: function changeAdmin(address newAdmin) public onlyAdmin {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L84

```solidity
186: function setDrips(
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L186

```solidity
220: function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L220







### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Avoid Floating Pragmas: The Version Should Be Locked

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### <ins>Proof Of Concept</ins>

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [AddressDriver.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L2

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [Caller.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L2

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [Drips.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L2

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [DripsHub.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L2

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [ImmutableSplitsDriver.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L2

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [Managed.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L2

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [NFTDriver.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L2

```solidity
Found usage of floating pragmas ^0.8.17 of Solidity in [Splits.sol]
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L2








### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

```solidity
84: function changeAdmin(address newAdmin) public onlyAdmin {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L84




#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.







### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

#### <ins>Proof Of Concept</ins>

All in-scope contracts




### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
156: balance.splittable = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L156

```solidity
188: balance.collectable = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L188





### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Imports can be grouped together

Consider importing OZ first, then all interfaces, then all utils.

#### <ins>Proof Of Concept</ins>


```solidity
12: import {Managed} from "./Managed.sol";
13: import {ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
14: import {SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L12-L14

```solidity
4: import {Drips, DripsConfig, DripsHistory, DripsConfigImpl, DripsReceiver} from "./Drips.sol";
5: import {Managed} from "./Managed.sol";
6: import {Splits, SplitsReceiver} from "./Splits.sol";
7: import {IERC20} from "openzeppelin-contracts/token/ERC20/IERC20.sol";
8: import {SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L4-L8

```solidity
4: import {DripsHub, SplitsReceiver, UserMetadata} from "./DripsHub.sol";
5: import {Managed} from "./Managed.sol";
6: import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L4-L6

```solidity
4: import {DripsHistory, DripsHub, DripsReceiver, SplitsReceiver, UserMetadata} from "./DripsHub.sol";
5: import {Managed} from "./Managed.sol";
6: import {Context, ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
7: import {IERC20, SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";
8: import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L4-L8







### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


All in-scope contracts


#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```

#### <ins>Proof Of Concept</ins>
For example, the following are missing natspec:

```solidity
function _call(address sender, address to, bytes memory data, uint256 value)
        internal
        returns (bytes memory returnData)
    {
        // Encode the message sender as per ERC-2771
        return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);
    }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L202

```solidity
    /// @notice Extracts dripId from a `DripsConfig`
    function dripId(DripsConfig config) internal pure returns (uint32) {
        return uint32(DripsConfig.unwrap(config) >> 224);
    }

    /// @notice Extracts amtPerSec from a `DripsConfig`
    function amtPerSec(DripsConfig config) internal pure returns (uint160) {
        return uint160(DripsConfig.unwrap(config) >> 64);
    }

    /// @notice Extracts start from a `DripsConfig`
    function start(DripsConfig config) internal pure returns (uint32) {
        return uint32(DripsConfig.unwrap(config) >> 32);
    }

    /// @notice Extracts duration from a `DripsConfig`
    function duration(DripsConfig config) internal pure returns (uint32) {
        return uint32(DripsConfig.unwrap(config));
    }

    /// @notice Compares two `DripsConfig`s.
    /// First compares their `amtPerSec`s, then their `start`s and then their `duration`s.
    function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {
        return DripsConfig.unwrap(config) < DripsConfig.unwrap(otherConfig);
    }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L72-L96


### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> No need to initialize uints to zero

There is no need to initialize `uint` variables to zero as their default value is `0`

#### <ins>Proof Of Concept</ins>

```solidity
478: uint256 idx = 0;
489: uint256 amt = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L478

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L489



```solidity
744: uint256 spent = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L744

```solidity
880: uint256 currIdx = 0;
881: uint256 newIdx = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L880-L881

```solidity
60: uint256 weightSum = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L60

```solidity
126: uint32 splitsWeight = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L126

```solidity
157: uint32 splitsWeight = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L157

```solidity
228: uint64 totalWeight = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L228






### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


```solidity
122: function setDrips(
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L122

```solidity
158: function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L158

```solidity
186: function setDrips(
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L186

```solidity
220: function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L220





### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
30: constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L30

```solidity
219: constructor(uint32 cycleSecs, bytes32 dripsStorageSlot)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L219

```solidity
109: constructor(uint32 cycleSecs_)
        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
        Splits(_erc1967Slot("eip1967.splits.storage"))
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L109

```solidity
28: constructor(DripsHub _dripsHub, uint32 _driverId)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L28

```solidity
73: constructor()
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L73

```solidity
158: constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0))
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L158

```solidity
32: constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
        ERC721("DripsHub identity", "DHI")
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L32

```solidity
94: constructor(bytes32 splitsStorageSlot)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L94





### <a href="#Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### <ins>Proof Of Concept</ins>
All in-scope contracts

For example, the following are missing natspec:

```solidity
function _call(address sender, address to, bytes memory data, uint256 value)
        internal
        returns (bytes memory returnData)
    {
        // Encode the message sender as per ERC-2771
        return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);
    }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L202

```solidity
    /// @notice Extracts dripId from a `DripsConfig`
    function dripId(DripsConfig config) internal pure returns (uint32) {
        return uint32(DripsConfig.unwrap(config) >> 224);
    }

    /// @notice Extracts amtPerSec from a `DripsConfig`
    function amtPerSec(DripsConfig config) internal pure returns (uint160) {
        return uint160(DripsConfig.unwrap(config) >> 64);
    }

    /// @notice Extracts start from a `DripsConfig`
    function start(DripsConfig config) internal pure returns (uint32) {
        return uint32(DripsConfig.unwrap(config) >> 32);
    }

    /// @notice Extracts duration from a `DripsConfig`
    function duration(DripsConfig config) internal pure returns (uint32) {
        return uint32(DripsConfig.unwrap(config));
    }

    /// @notice Compares two `DripsConfig`s.
    /// First compares their `amtPerSec`s, then their `start`s and then their `duration`s.
    function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {
        return DripsConfig.unwrap(config) < DripsConfig.unwrap(otherConfig);
    }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L72-L96



#### <ins>Recommended Mitigation Steps</ins>

NatSpec comments should be increased in contracts











### <a href="#Summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
207: return Address.functionCallWithValue(to, abi.encodePacked(data, sender)

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L207




#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 



### <a href="#Summary">[NC&#x2011;13]</a><a name="NC&#x2011;13"> Use of Block.Timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
References: SWC ID: 116

#### <ins>Proof Of Concept</ins>


```solidity
173: require(block.timestamp <= deadline, "Execution deadline expired");
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L173



#### <ins>Recommended Mitigation Steps</ins>
Block timestamps should not be used for entropy or generating random numbers—i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.



