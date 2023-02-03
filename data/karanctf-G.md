## [G-1]
Use proper viarable packing for state vairables to save gas in contract `DripsHub.sol#54`
In below code snipet use to save gas as Each storage slot in a smart contract costs gas. To save gas, Solidity contracts can be arranged so that multiple variables fit in a single slot. This strategy is known as variable packing and when packed variables are used together it can drastically reduce gas costs.
```solidity
uint8
uint32
uint256
uint256
uint256
uint256
uint256
bytes32
```

```diff
/// @notice Maximum number of drips receivers of a single user.
    /// Limits cost of changes in drips configuration.
    uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;
    /// @notice The additional decimals for all amtPerSec values.
    uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
    /// @notice The multiplier for all amtPerSec values.
    uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
    /// @notice Maximum number of splits receivers of a single user. Limits the cost of splitting.
    uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
    /// @notice The total splits weight of a user
    uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
    /// @notice The offset of the controlling driver ID in the user ID.
    /// In other words the controlling driver ID is the highest 32 bits of the user ID.
    uint256 public constant DRIVER_ID_OFFSET = 224;
    /// @notice The total amount the contract can store of each token.
    /// It's the minimum of _MAX_TOTAL_DRIPS_BALANCE and _MAX_TOTAL_SPLITS_BALANCE.
    uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;
    /// @notice The ERC-1967 storage slot holding a single `DripsHubStorage` structure.
    bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage");
```

## [G-2]
Functions guaranteed to revert when called by normal users can be marked
- Function modifiers prevent unauthorized users from executing the function and revert if necessary.
- Marking a function as payable lowers gas costs and deployment cost by avoiding checks for payment.

```solidity
Managed.sol:84:    function changeAdmin(address newAdmin) public onlyAdmin {
Managed.sol:90:    function grantPauser(address pauser) public onlyAdmin {
Managed.sol:97:    function revokePauser(address pauser) public onlyAdmin {
Managed.sol:122:    function pause() public onlyAdminOrPauser whenNotPaused {
Managed.sol:128:    function unpause() public onlyAdminOrPauser whenPaused {
Managed.sol:151:    function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {
```
