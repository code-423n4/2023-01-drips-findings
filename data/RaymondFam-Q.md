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
