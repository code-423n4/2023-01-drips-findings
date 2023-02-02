## Summary
_This report does not include publicly known issues_
### Gas Optimizations
| |Issue|Instances|
|-|:-|:-:|
| [GAS-01] | Storage slot can be precomputed to be `constant`| 8 | 
| [GAS-02] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 3 | 
| [GAS-03] | `++i/i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops | 22 | 
| [GAS-04] | Setting the `constructor` to `payable` | 7 |
| [GAS-05] | Functions guaranteed to revert when called by normal users can be marked `payable` | 11 |
| [GAS-06] | Usage of `uint`/`int` smaller than 32 bytes | 2 |
| [GAS-07] | If-Else unnecessary | 1 |
| [GAS-08] | Stack variable used as a cheaper cache for a state variable is only used once | 1 |
| [GAS-09] | State variables can be packed into fewer storage slots | 2 |

Total: 57 instances over 9 issues


## Gas Optimizations
### [GAS-01] Storage slot can be precomputed to be `constant`
`_erc1967Slot(...)` can be computed before deployment and state variables `XXXStorageSlot` can be `constant` to save deployment gas.
```solidity
File: DripsHub.sol
L72:		bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage");
L110:		Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
L111:		Splits(_erc1967Slot("eip1967.splits.storage"))
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)
```solidity
File: Drips.sol
L119:		bytes32 private immutable _dripsStorageSlot;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)
```solidity
File: Splits.sol
L27:		bytes32 private immutable _splitsStorageSlot;)
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol)
```solidity
File: Managed.sol
L20:		bytes32 private immutable _managedStorageSlot = _erc1967Slot("eip1967.managed.storage");)
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol)
```solidity
File: NFTDriver.sol
L27:		bytes32 private immutable _mintedTokensSlot = _erc1967Slot("eip1967.nftDriver.storage");
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol)

```solidity
File: ImmutableSplitsDriver.sol
L19:		bytes32 private immutable _counterSlot = _erc1967Slot("eip1967.immutableSplitsDriver.storage");
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol)

Note that the function `_erc1967Slot(string memory name) //File: Managed.sol#L136` can then be removed.

### [GAS-02] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/MiniGlome/f462d69a30f68c89175b0ce24ce37cae)**
Same for `-=`, `*=` and `/=`.
```solidity
File: DripsHub.sol
L632:		totalBalances[erc20] += amt;
L636:		_dripsHubStorage().totalBalances[erc20] -= amt;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)
```solidity
File: Splits.sol
L99:		 _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L99)

### [GAS-03] `++i/i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP
```solidity
File: Splits.sol
L127:		for (uint256 i = 0; i < currReceivers.length; i++) {
L158:		for (uint256 i = 0; i < currReceivers.length; i++) {
L231:		for (uint256 i = 0; i < receivers.length; i++) {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol)
```solidity
File: NFTDriver.sol
L93:		StorageSlot.getUint256Slot(_mintedTokensSlot).value++;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol)
```solidity
File: ImmutableSplitsDriver.sol
L59:		StorageSlot.getUint256Slot(_counterSlot).value++;
L61:		for (uint256 i = 0; i < receivers.length; i++) {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol)
```solidity
File: DripsHub.sol
L613:		for (uint256 i = 0; i < userMetadata.length; i++) {
L136:		driverId = dripsHubStorage.nextDriverId++;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)
```solidity
File: Drips.sol
L247:		for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
L287:		for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
L358:		for (uint256 i = 0; i < squeezedNum; i++) {
L422:		for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
L428:		squeezedRevIdxs[squeezedNum++] = i;
L450:		for (uint256 i = 0; i < dripsHistory.length; i++) {
L490:		for (; idx < receivers.length; idx++) {
L563:		for (uint256 i = 0; i < receivers.length; i++) {
L655:		state.currCycleConfigs++;
L664:		for (uint256 i = 0; i < newReceivers.length; i++) {
L959:		currIdx++;
L962:		newIdx++;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)
```solidity
File: Caller.sol
L174:		uint256 currNonce = nonce[sender]++;
L196:		for (uint256 i = 0; i < calls.length; i++) {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol)

### [GAS-04]  Setting the `constructor` to `payable`
Saves ~13 gas per instance
```solidity
File: Drips.sol
L219:		 constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219)
```solidity
File: DripsHub.sol
L109:		constructor(uint32 cycleSecs_)
        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
        Splits(_erc1967Slot("eip1967.splits.storage"))
    {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109)
```solidity
File: AddressDriver.sol
L30:		constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
    {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30)
```solidity
File: ImmutableSplitsDriver.sol
L28:		constructor(DripsHub _dripsHub, uint32 _driverId) {
        dripsHub = _dripsHub;
        driverId = _driverId;
        totalSplitsWeight = _dripsHub.TOTAL_SPLITS_WEIGHT();
    }
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28)
```solidity
File: NFTDriver.sol
L32:		constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
        ERC721("DripsHub identity", "DHI")
    {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32)
```solidity
File: Managed.sol
L158:		constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0)) {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L158)
```solidity
File: Splits.sol
L94:		constructor(bytes32 splitsStorageSlot) {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94)

### [GAS-05] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
```solidity
File: Managed.sol
L84:		function changeAdmin(address newAdmin) public onlyAdmin {
L90:		function grantPauser(address pauser) public onlyAdmin {
L97:		function revokePauser(address pauser) public onlyAdmin {
L122:		function pause() public onlyAdminOrPauser whenNotPaused {
L128:		function unpause() public onlyAdminOrPauser whenPaused {
L151:		function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol)
```solidity
File: DripsHub.sol
L386:		function collect(uint256 userId, IERC20 erc20)
        public
        whenNotPaused
        onlyDriver(userId)
        returns (uint128 amt)
    {
L409:		function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)
        public
        whenNotPaused
        onlyDriver(userId)
    {
L510:		function setDrips(
        uint256 userId,
        IERC20 erc20,
        DripsReceiver[] memory currReceivers,
        int128 balanceDelta,
        DripsReceiver[] memory newReceivers,
        // slither-disable-next-line similar-names
        uint32 maxEndHint1,
        uint32 maxEndHint2
    ) public whenNotPaused onlyDriver(userId) returns (int128 realBalanceDelta) {
L576:		function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
        public
        whenNotPaused
        onlyDriver(userId)
    {
L608:		function emitUserMetadata(uint256 userId, UserMetadata[] calldata userMetadata)
        public
        whenNotPaused
        onlyDriver(userId)
    {
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)

### [GAS-06] Usage of `uint`/`int` smaller than 32 bytes
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.
```solidity
File: Drips.sol
L108:		uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L108)
```solidity
File: DripsHub.sol
L58:		uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L58)

### [GAS-07] If-Else unnecessary
`if (a > b) { return false }` ==> `return !(a > b)`
```solidity
File: Drips.sol
L756:		if (spent > balance) {
                    return false;
                }
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L756)

### [Gas-08] Stack variable used as a cheaper cache for a state variable is only used once
The variable `splitsStates` is used only once, thus it is not useful and should be inlined to save gas.
```solidity
File: Splits.sol
L148:		mapping(uint256 => SplitsState) storage splitsStates = _splitsStorage().splitsStates;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L148)

### [GAS-09] State variables can be packed into fewer storage slots
State variables can be packed in slot of 32 bytes to reduce storage usage.
See more: https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html
```solidity
File: Drips.sol

// Before packing
uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;
uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
uint32 internal immutable _cycleSecs;
bytes32 private immutable _dripsStorageSlot;

// After packing
uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;
uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
bytes32 private immutable _dripsStorageSlot;
uint32 internal immutable _cycleSecs;
uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)
```solidity
File: DripsHub.sol

// Before packing
uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;
uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
uint256 public constant DRIVER_ID_OFFSET = 224;
uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;
bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage");

// After packing
uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;
uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
uint256 public constant DRIVER_ID_OFFSET = 224;
uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;
bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage");
uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
```
[Link to code](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)