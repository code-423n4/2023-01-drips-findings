# [L-01] Unsafe critical address change
## Targets
- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84-L86
## Impact
In case an incorrect admin address is set (due to a typo or due to setting the zero address, which is the default value of the address type), any contract that inherits from `Managed` (`AddressDriver`, `Caller`, `DripsHub`, `ImmutableSplitsDriver`, `NFTDriver`) may end up in the unmanageable state. Since admin is the only role that's allowed to upgrade contracts, there will be no way to fix a broken contract.
## Proof of Concept
All contracts of the project inherit from the [Managed](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L18) contract, which adds admin privileges to the contracts. However, changing of an admin is implemented as a [one-step function](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84-L86):
```solidity
function changeAdmin(address newAdmin) public onlyAdmin {
    _changeAdmin(newAdmin);
}
```
In case a new admin is set to a wrong address (due to a typo, a miscommunication, or a software bug), there will not be a way to revert the change.
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider using a 2-step process to change admins of the contract. As a reference, consider [this implementation by OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/91e8d0ba3c3beb8a1db31310d8599664e48639ef/contracts/access/Ownable2Step.sol).



# [L-02] Incomplete information in an event
## Targets
- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L615
## Impact
The `UserMetadataEmitted` event may be hard to find in the logs of a node, which may break some off-chain integrations (e.g. analytics tools). Also, the fact than the event can be emitted by a driver unconditionally, can make it much harder to find a needed event.
## Proof of Concept
The `DripsHub` contract allows drivers to unconditionally emit the `UserMetadataEmitted` event, via the [emitUserMetadata](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L608) function. The event has the following fields:
```solidity
event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);
```

Since the `emitUserMetadata` function can be called by any driver, the logs of the contract can be clogged with `UserMetadataEmitted` events emitted by different drivers, and it'll be hard to find events emitted by a specific driver (Ethereum nodes allow quick search only on indexed fields). Consider this example:
1. A project decides to use Drips to distribute payments to its users.
1. The project has created its own driver. The driver calls the `emitUserMetadata` function to emit custom user data that's analyzed off-chain.
1. Since `UserMetadataEmitted` events are not indexed by driver IDs, the project's developers will have to scan the entire transactions history of the `DripsHub` contract to finds events emitted by their driver.
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider adding the indexed `driverId` field to the `UserMetadataEmitted` event so that projects that use Drips could find events emitted by their drivers.