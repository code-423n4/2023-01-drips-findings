| Total Low issues   |
|--------------------|

| Risk   | Issues Details                                                | Number        |
|--------|---------------------------------------------------------------|---------------|
| [L-01] | The protocol is using `draft-EIP712.sol` which is deprecated  | 1             |
| [L-02] | Integer overflow by unsafe casting                            | 11            |
| [L-03] | Use `safeMint` instead of `mint` for ERC721                   | 1             |
| [L-04] | Array lengths not checked                                     | 4             |
| [L-05] | Critical changes should use two-step procedure                | 1             |
| [L-06] | Loss of precision due to rounding                             | 4             |
| [L-07] | No storage gap for upgradeable contracts                      | 1             |
| [L-08] | Unhandled return values of `transferFrom`                     | 1             |
| [L-09] | Missing Event for initialize                                  | 8             |
| [L-10] | `admin` can renounce while system is paused                   | 1             |

| Total Non-Critical issues   |
|-----------------------------|

| Risk    | Issues Details                                                                       | Number        |
|---------|--------------------------------------------------------------------------------------|---------------|
| [NC-01] | Lock pragmas to specific compiler version                                            | All Contracts |
| [NC-02] | Contracts should have full test coverage                                             | All Contracts |
| [NC-03] | Constants in comparisons should appear on the left side                              | 12            |
| [NC-04] | `Address(0)` checks                                                                  | 4             |
| [NC-05] | Include `@return` parameters in NatSpec comments                                     | 3 Contracts   |
| [NC-06] | Function writing does not comply with the `Solidity Style Guide`                     | All Contracts |
| [NC-07] | Use `bytes.concat()` and `string.concat()`                                           | 1             |
| [NC-08] | For functions, follow Solidity standard naming conventions                           | All Contracts |
| [NC-09] | Generate perfect code headers every time                                             | All Contracts |
| [NC-10] | There is no need to cast a variable that is already an address, such as `address(x)` | 2             |
| [NC-11] | Consider using `delete` rather than assigning zero to clear values                   | 2             |
| [NC-12] | Constants should be defined rather than using magic numbers	                         | 2             |
| [NC-13] | Need Fuzzing test                                                                    | 1 Contracts   |
| [NC-14] | Assembly Codes Specific – Should Have Comments                                       | 6             |
| [NC-15] | Use SMTChecker                                                                       |               |
| [NC-16] | Add NatSpec comment to `mapping`                                                     | 9             |
| [NC-17] | Not using the named return variables anywhere in the function is confusing           | All Contracts |

## [L-01] The protocol is using `draft-EIP712.sol` which is deprecated

#### Description

While OpenZeppelin draft contracts are safe to use and have been audited, their 'draft' status means that the EIPs they're based on are not finalized, and thus there may be breaking changes in even [minor releases](https://docs.openzeppelin.com/contracts/3.x/api/drafts). If a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to has breaking changes in the new version, you'll encounter unnecessary delays in porting and testing replacement contracts. Ensure that you have extensive test coverage of this area so that differences can be automatically detected, and have a plan in place for how you would fully test a new version of these contracts if they do indeed change unexpectedly.

```solidity
// EIP-712 is Final as of 2022-08-11. This file is deprecated.
```

> [Refrence](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/draft-EIP712.sol#L6)

#### Lines of code 

- [Caller.sol#L5](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L5)

#### Recommended Mitigation Steps

Consider creating a forked version of the file rather than importing it from the package, and manually patch your fork as changes are made.

## [L-02] Integer overflow by unsafe casting

#### Description

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

```solidity
        splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);
```

#### Lines of code 

- [Drips.sol#L497](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L497)
- [Drips.sol#L692](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L692)
- [Drips.sol#L697](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L697)
- [Drips.sol#L997](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L997)
- [Drips.sol#L1012](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1012)
- [Drips.sol#L1053](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1053)
- [Drips.sol#L1054](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1054)
- [Drips.sol#L1129](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1129)
- [DripsHub.sol#L119](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L119)
- [Splits.sol#L130](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L130)
- [Splits.sol#L161](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L161)

#### Recommended Mitigation Steps

Use openzeppelin [`safeCast.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol) library to prevent unexpected overflows when casting from uint256.

## [L-03] Use `safeMint` instead of `mint` for ERC721

#### Description

Users could lost their NFTs if `msg.sender` is a contract address that does not support `ERC721`, the NFT can be frozen in the contract forever.

As per the documentation of EIP-721:

> A wallet/broker/auction application MUST implement the wallet interface if it will accept safe transfers.

> Ref: https://eips.ethereum.org/EIPS/eip-721

As per the documentation of `ERC721.sol` by Openzeppelin:

> Ref: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L274-L285

```solidity
    function mint(address to, UserMetadata[] calldata userMetadata)
        public
        whenNotPaused
        returns (uint256 tokenId)
    {
        tokenId = _registerTokenId();
        _mint(to, tokenId);
        if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
    }
```

#### Lines of code 

- [NFTDriver.sol:68](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L68)

#### Recommended Mitigation Steps

Use [`_safeMint`](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC721.sol#L193-L202) instead of [`mint`](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC721.sol#L157-L170) to check received address support for ERC721 implementation.

## [L-04] Array lengths not checked

#### Description

If the length of the arrays are not required to be of the same length, user operations may not be fully executed.

```solidity
    function setDrips(
        IERC20 erc20,
        DripsReceiver[] calldata currReceivers,
        int128 balanceDelta,
        DripsReceiver[] calldata newReceivers,
        // slither-disable-next-line similar-names
        uint32 maxEndHint1,
        uint32 maxEndHint2,
        address transferTo
    ) public whenNotPaused returns (int128 realBalanceDelta) {
        // @audit array length
        if (balanceDelta > 0) {
            _transferFromCaller(erc20, uint128(balanceDelta));
        }
        realBalanceDelta = dripsHub.setDrips(
            callerUserId(),
            erc20,
            currReceivers,
            balanceDelta,
            newReceivers,
            maxEndHint1,
            maxEndHint2
        );
        if (realBalanceDelta < 0) {
            // @audit check if transferTo is a valid address
            erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
        }
    }
```

#### Lines of code 

- [AddressDriver.sol#L122](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L122)
- [DripsHub.sol#L510](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L510)
- [NFTDriver.sol#L186](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L186)
- [Drips.sol#L610](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L610)

#### Recommended Mitigation Steps

```solidity
if (currReceivers != newReceivers) revert("Array length mismatch");
```

## [L-05] Critical changes should use two-step procedure

#### Description

The protocol inherit openzeppelin [`ERC1967Upgrade.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/ERC1967/ERC1967Upgrade.sol) which does not use two-step procedure when changing the `admin`, and since the protocol rely heavily on the [`onlyAdmin()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L46) modifier (8 results in 1 files), Thus using two-step procedure is a best practice
in case of transferring the `admin` role to an invalid address.

```solidity
    function changeAdmin(address newAdmin) public onlyAdmin {
        _changeAdmin(newAdmin);
    }
```

#### Lines of code 

- [Managed.sol#L84](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84)

#### Recommended Mitigation Steps

Consider adding two step procedure on the critical functions where the first is announcing a `pendingAdmin` and the new address should then claim the `admin` role.

## [L-06] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

```solidity
        splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);
```

#### Lines of code 

- [Splits.sol#L130](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L130)
- [Splits.sol#L161](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L161)
- [Drips.sol#L1045](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1045)
- [Drips.sol#L1047](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1047)

## [L-07] No storage gap for upgradeable contracts

#### Description

For upgradeable contracts, inheriting contracts may introduce new variables. In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) for a description of this storage variable. 

#### Lines of code 

- [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol)

#### Recommended Mitigation Steps

Consider adding a storage gap at the end of the upgradeable contract:

```solidity
  /**
   * @dev This empty reserved space is put in place to allow future versions to add new
   * variables without shifting down storage in the inheritance chain.
   * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
   */
  uint256[50] private __gap;
```

## [L-08] Unhandled return values of `transferFrom`   

#### Description

ERC20 implementations are not always consistent. Some implementations of `transferFrom` could return false on failure instead of reverting. It is safer to check the return value and revert on false to prevent these failures.

This is a known issue with solmate and Opanzeppelin ERC20 libraries, For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens, hence Tether (USDT) `transferFrom()` function do not return boolean as the specification requires, and instead have no return value.

```solidity
    function transferFrom(address from, address to, uint256 tokenId)
        public
        override
        whenNotPaused
    {
        super.transferFrom(from, to, tokenId);
    }
```

#### Lines of code 

- [NFTDriver.sol#L282](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L282)

#### Recommended Mitigation Steps

Check the return value and `revert()` on false.

## [L-09] Missing Event for initialize

#### Description

Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip.

#### Lines of code 

- [AddressDriver.sol#L30-L35](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30-L35)
- [Drips.sol#L219-L223](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219-L223)
- [DripsHub.sol#L109-L114](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109-L114)
- [ImmutableSplitsDriver.sol#L28-L32](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28-L32)
- [Managed.sol#L73-L75](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L73-L75)
- [Managed.sol#L158-L160](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L158-L160)
- [NFTDriver.sol#L32-L38](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32-L38)
- [Splits.sol#L94-L96](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94-L96)

#### Recommended Mitigation Steps

Add Event-Emit

## [L-10] `admin` can renounce while system is paused

#### Description

The contract `admin` is not prevented from renouncing the role/ownership while the contract is paused, which would cause any user assets stored in the protocol, to be locked indefinitely.

```solidity
    function pause() public onlyAdminOrPauser whenNotPaused {
        _managedStorage().isPaused = true;
        emit Paused(msg.sender);
    }
```

#### Lines of code 

- [Managed.sol#L122-L125](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L122-L125)

## [NC-01] Lock pragmas to specific compiler version

#### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. 

> Ref: https://swcregistry.io/docs/SWC-103

```solidity
pragma solidity ^0.8.17;
```

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-drips/tree/main/src)

#### Recommended Mitigation Steps

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. 

> https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

## [NC-02] Contracts should have full test coverage

#### Description

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

```solidity
- What is the overall line coverage percentage provided by your tests?:  90
```

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-drips/tree/main/src)

#### Recommended Mitigation Steps

Line coverage percentage should be 100%.

## [NC-03] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
        if (start == 0) {
            start = updateTime;
        }
```

#### Lines of code 

- [AddressDriver.sol#L173](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L173)
- [DDrips.sol#L322](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L322)
- [Drips.sol#L454](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L454)
- [Drips.sol#L691](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L691)
- [Drips.sol#L839](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L839)
- [Drips.sol#L951](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L951)
- [Drips.sol#L994](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L994)
- [NFTDriver.sol#L288](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L288)
- [Splits.sol#L123](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L123)
- [Splits.sol#L152](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L152)
- [Splits.sol#L275](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L275)

#### Recommended Mitigation Steps

```solidity
        if (0 == start) {
            start = updateTime;
        }
```

## [NC-04] `Address(0)` checks 

#### Description

Check of `address(0)` to protect the code from `(0x0000000000000000000000000000000000000000)` address problem just in case. This is best practice or instead of suggesting that they verify `_address != address(0)`, you could add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

```solidity
    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
    {
        dripsHub = _dripsHub;
        driverId = _driverId;
    }
```

#### Lines of code 

- [AddressDriver.sol#L30-L35](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30-L35)
- [Drips.sol#L219-L223](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219-L223)
- [ImmutableSplitsDriver.sol#L28-L32](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28-L32)
- [NFTDriver.sol#L32-L38](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32-L38)

#### Recommended Mitigation Steps

Add a check for `address(0)`.

## [NC-05] Include `@return` parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `/// @return`. Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

- [Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)
- [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol)
- [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol)

#### Recommended Mitigation Steps

Include the `@return` argument in the NatSpec comments.

## [NC-06] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external` / `public` / `internal` / `private`
- `view` / `pure`

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-drips/tree/main/src)

#### Recommended Mitigation Steps

Follow [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html?highlight=Style#style-guide).

## [NC-07] Use `bytes.concat()` and `string.concat()`

#### Description

- `bytes.concat()` vs `abi.encodePacked(<bytes>,<bytes>)`
- `string.concat()` vs `abi.encodePacked(<string>,<string>)`

> https://docs.soliditylang.org/en/v0.8.17/types.html?highlight=bytes.concat#the-functions-bytes-concat-and-string-concat

```solidity
        return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);
```

#### Lines of code 

- [Caller.sol#L207](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L207)

#### Recommended Mitigation Steps

Use `bytes.concat()` and `string.concat()` instead.

## [NC-08] For functions, follow Solidity standard naming conventions

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-drips/tree/main/src)

#### Recommended Mitigation Steps

Follow solidity standard naming convention.

## [NC-09] Generate perfect code headers every time

#### Description

I recommend using header for Solidity code layout and readability.

> Ref: https://github.com/transmissions11/headers

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-drips/tree/main/src)

#### Recommended Mitigation Steps

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [NC-10] There is no need to cast a variable that is already an address, such as `address(x)`

#### Description

There is no need to cast a variable that is already an `address`, such as `address(x)`, `x` is also `address`.

```solidity
       if (erc20.allowance(address(this), address(dripsHub)) == 0) {
            erc20.safeApprove(address(dripsHub), type(uint256).max);
        }
```

#### Lines of code 

- [AddressDriver.sol#L173-L175](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L173-L175)
- [NFTDriver.sol#L288-L290](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L288-L290)

#### Recommended Mitigation Steps

```solidity
       if (erc20.allowance(address(this), dripsHub) == 0) {
            erc20.safeApprove(dripsHub, type(uint256).max);
        }
```

## [NC-11] Consider using `delete` rather than assigning zero to clear values

#### Description

The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

```solidity
        balance.splittable = 0;
```

#### Lines of code 

- [Splits.sol#L156](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L156)
- [Splits.sol#L188](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L188)

#### Recommended Mitigation Steps

```solidity
        delete balance.splittable;
```

## [NC-12] Constants should be defined rather than using magic numbers

#### Description

```solidity
            uint256 idxMid = (idx + idxCap) / 2;
```

```solidity
                uint256 end = (enoughEnd + notEnoughEnd) / 2;
```

#### Lines of code 

- [Drips.sol#L480](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L480)
- [Drips.sol#L717](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L717)

#### Recommended Mitigation Steps

```solidity
uint8 private constant TWO = 2;
```

## [NC-13] Need Fuzzing test

#### Description

In total 1 contract, 6 `unchecked{}` are used, the functions used are critical. For this reason, there must be fuzzing tests in the tests of the contract. Not seen in tests.

#### Lines of code 

- [Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)

#### Recommended Mitigation Steps

Use should fuzzing test like Echidna. As Alberto Cuesta Canada said: Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.

> Ref: https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

## [NC-14] Assembly Codes Specific – Should Have Comments

#### Description

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

#### Lines of code 

- [Drips.sol#L822-L824](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L822-L824)
- [Drips.sol#L1103-L1114](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1103-L1114)
- [Drips.sol#L1145-L1147](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1145-L1147)
- [DripsHub.sol#L624-L626](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L624-L626)
- [Managed.sol#L145-L147](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L145-L147)
- [Splits.sol#L286-L288](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L286-L288)

#### Recommended Mitigation Steps

Include NatSpec in assembly codes.

## [NC-15] Use SMTChecker

#### Description

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

> Ref: https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-drips/tree/main/src)

#### Recommended Mitigation Steps

Use SMTChecker

## [NC-16] Add NatSpec comment to `mapping`

#### Description

Add NatSpec comments describing mapping keys and values.

```solidity
    mapping(address => uint256) public nonce;
```

#### Lines of code 

- [Caller.sol#L88](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L88)
- [Caller.sol#L90](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L90)
- [Drips.sol#L176](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L176)
- [Drips.sol#L185](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L185)
- [Drips.sol#L203](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L203)
- [DripsHub.sol#L99](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L99)
- [DripsHub.sol#L101](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L101)
- [Splits.sol#L76](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L76)
- [Splits.sol#L83](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L83)

#### Recommended Mitigation Steps

```solidity
 /// @dev  address(user) => uint256(nonce)
    mapping(address => uint256) public nonce;
```

## [NC-17] Not using the named return variables anywhere in the function is confusing

#### Description

Not using the named return variables anywhere in the function is confusing.

```solidity
    function isAuthorized(address sender, address user) public view returns (bool authorized) {
        return _authorized[sender].contains(user);
    }
```

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-drips/tree/main/src)

#### Recommended Mitigation Steps

Consider changing the variable to be an unnamed one.

```solidity
    function isAuthorized(address sender, address user) public view returns (bool) {
        return _authorized[sender].contains(user);
    }
```
