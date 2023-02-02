### QA Issues List
| Number |Issues Details
|:--:|:-------|:--:|
|[QA-01]| Use latest Solidity version
|[QA-02]| Use stable pragma statement
|[QA-03]| Initial value check is missing in setter functions
|[QA-04]|Draft OpenZeppelin dependencies
|[QA-05]|Missing zero address validation
|[QA-06]| Missing event for critical parameter change
|[QA-07]|If the `mint()` function will not be used it is better to delete it
|[QA-08]|Single-step ownership transfer can be dangerous
***

## [QA-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version - `0.8.18`. Currently the solidity version in contracts is 0.8.17 which was found to possess some bugs.
***

## [QA-02] Use stable pragma statement

Using a floating pragma statement `pragma solidity ^0.8.17;` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.
***

## [QA-03] INITIAL VALUE CHECK IS MISSING IN SETTER FUNCTIONS
Setter functions are missing checks to validate if the new value being set is the same as the current value already set in the contract. Such checks will showcase mismatches between on-chain and off-chain states.
```
DripsHub.sol
152: function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
510: function setDrips(
576: function setSplits(uint256 userId, SplitsReceiver[] memory receivers)

Managed.sol
84: function changeAdmin(address newAdmin) public onlyAdmin {
```
Checking whether the current value and the new value are the same should be added.
***

## [QA-04] DRAFT OPENZEPPELIN DEPENDENCIES
The `Caller.sol` contract utilised `draft-EIP712.sol`. This contract are still a draft and are not considered ready for mainnet use.

OpenZeppelin contracts may be considered draft contracts if they have not received adequate security auditing or are liable to change with future development.
```
5: import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";
```
***
## [QA-05] Missing zero address validation

Setters of address type parameters should include a zero-address check otherwise contract functionality may become inaccessible or tokens burnt forever.

```
DripsHub.sol
134: function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
152: function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {

NFTDriver.sol:
62: function mint(address to, UserMetadata[] calldata userMetadata)
function safeMint(address to, UserMetadata[] calldata userMetadata)
79: function safeMint(address to, UserMetadata[] calldata userMetadata)
109: function collect(uint256 tokenId, IERC20 erc20, address transferTo)
186: function setDrips(

Managed.sol
84: function changeAdmin(address newAdmin) public onlyAdmin {
90: function grantPauser(address pauser) public onlyAdmin {

AddressDriver.sol:
60: function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
```

#### Recommendation

Check that the address is not zero.
***

## [QA-06] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.
```
DripsHub.sol
386: function collect(uint256 userId, IERC20 erc20)
409: function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)
510: function setDrips(

NFTDriver.sol
79: function safeMint(address to, UserMetadata[] calldata userMetadata)
109: function collect(uint256 tokenId, IERC20 erc20, address transferTo)
133: function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)
186: function setDrips(

Managed.sol
84: function changeAdmin(address newAdmin) public onlyAdmin {

AddressDriver.sol:
60: function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
```
***

## [QA-07] If the `mint()` function will not be used it is better to delete it
```
NFTDriver.sol

/// @notice Mints a new token controlling a new user ID and transfers it to an address.
    /// Emits user metadata for the new token.
    /// Usage of this method is discouraged, use `safeMint` whenever possible.
    /// @param to The address to transfer the minted token to.
    /// @param userMetadata The list of user metadata to emit for the minted token.
    /// The keys and the values are not standardized by the protocol, it's up to the user
    /// to establish and follow conventions to ensure compatibility with the consumers.
    /// @return tokenId The minted token ID. It's equal to the user ID controlled by it.
62:  function mint(address to, UserMetadata[] calldata userMetadata)
        public
        whenNotPaused
        returns (uint256 tokenId)
    {
        tokenId = _registerTokenId();
        _mint(to, tokenId);
        if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
    }
```
***

## [QA-08] Single-step ownership transfer can be dangerous

## Summary
The `changeAdmin` function is a delicate process. It could lead to loss of authorization to critical functions in case of typos or bad copy/paste. A two step process should be used as a guard against setting the wrong admin.

Single-step ownership transfer means that if a wrong address was passed when transferring ownership or admin rights it can mean that role is lost forever. 

## Impact
The owner is responsible for setting multiple critical operation. Loss of the ownership role would therefore lead to breaking of how the protocol works including loss of funds.

## Code Snippet

```
Managed.sol

84: function changeAdmin(address newAdmin) public onlyAdmin {
85:        _changeAdmin(newAdmin);
86:    }
```

## Recommendation
The new owner should be added without first overwriting the previous one. Once this is done, the new owner can then remove the old one.