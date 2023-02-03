## Interfaces and libraries should be separately saved and imported
Some contracts have interface(s) or libraries showing up in its/their entirety at the top/bottom of the contract facilitating an ease of references on the same file page. This has, in certain instances, made the already large contract size to house an excessive code base. Additionally, it might create difficulty locating them when attempting to cross reference the specific interfaces embedded elsewhere but not saved into a particular .sol file.

Consider saving the interfaces and libraries entailed respectively, and having them imported like all other files.

Here are some of the instances entailed:

[File: Drips.sol#L49-L97](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L49-L97)

## Solidity's Style Guide on contract layout
According to Solidity's Style Guide below:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the view and pure functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Consider adhering to the above guidelines for all contract instances entailed.

## Return statement on named returns

Functions with named returns should not have a return statement to avoid unnecessary confusion.

For instance, the following `_balanceAt()` may be refactored as:

[File: Drips.sol#L533-L543](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L533-L543)

```diff
    function _balanceAt(
        uint256 userId,
        uint256 assetId,
        DripsReceiver[] memory receivers,
        uint32 timestamp
    ) internal view returns (uint128 balance) {
        DripsState storage state = _dripsStorage().states[assetId][userId];
        require(timestamp >= state.updateTime, "Timestamp before last drips update");
        require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list");
-        return _balanceAt(state.balance, state.updateTime, state.maxEnd, receivers, timestamp);
+        balance = _balanceAt(state.balance, state.updateTime, state.maxEnd, receivers, timestamp);
    }
```
## Use `delete` to clear variables
`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address or a boolean to false. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at x.

The delete key better conveys the intention and is also more idiomatic.

For instance, the `a = false` instance below may be refactored as follows:

[File: Managed.sol#L129](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L129)

```diff
-        _managedStorage().isPaused = false;
+        delete _managedStorage().isPaused;
```
## Unspecific compiler version pragma
For some source-units the compiler version pragma is very unspecific, i.e. ^0.8.17. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Make use of scientific notation
Constants entailing huge numerals could adopt scientific notation for better readability while minimizing human input error.

For instance, the specific instance below may be refactored as:

[File: Drips.sol#L110](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L110)

```diff
-    uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
+    uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1e9;
```
## Sanity checks at the constructor
Adequate zero address and zero value checks should be implemented at the constructor to avoid accidental error(s) that could result in non-functional calls associated with it particularly when assigning immutable variables.

Here is a specific instance entailed:

[File: DripsHub.sol#L109-L114](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109-L114)

```diff
    constructor(uint32 cycleSecs_)
        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
        Splits(_erc1967Slot("eip1967.splits.storage"))
    {
+        require(cycleSecs_ != 0, "Zaro value input");
        return;
    }
```
## Early return in constructor forbidden if child contract has immutable fields
According to the link below: 

https://github.com/ethereum/solidity/issues/12379

The constructor may fail to compile if it contains an return statement and another contract containing immutable fields inherits from it.

Although no other contracts in the protocol are inheriting from DripsHub.sol, care must be given for any future instances that might do so.

[File: DripsHub.sol#L109-L114](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109-L114) 

```solidity
    constructor(uint32 cycleSecs_)
        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
        Splits(_erc1967Slot("eip1967.splits.storage"))
    {
        return;
    }
```
## Loss of precision due to rounding by big division
Consider scaling up the numerator(s) if need be when entailing a a big divisor (denominator), e.g. _AMT_PER_SEC_MULTIPLIER = 1_000_000_000 in Drips.sol, to minimize rounding issues.

Here are the two specific instances entailed:

[File: Drips.sol#L1044-L1047](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1044-L1047)

```solidity
            int256 amtPerSecMultiplier = int256(_AMT_PER_SEC_MULTIPLIER);
            int256 fullCycle = (int256(uint256(_cycleSecs)) * amtPerSec) / amtPerSecMultiplier;
            // slither-disable-next-line weak-prng
            int256 nextCycle = (int256(timestamp % _cycleSecs) * amtPerSec) / amtPerSecMultiplier;
```
