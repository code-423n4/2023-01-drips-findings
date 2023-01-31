# Gas Optimizations Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | x += y or x -= y costs more gas than x = x + y or x = x - y for state variables                                                    |     5     |
| [G-002] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |     1     |
| [G-003] | Functions guaranteed to revert when called by normal users can be marked payable                                                   |     5     |
| [G-004] | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate                 |     2     |
| [G-005] | Using `calldata` instead of memory for read-only arguments in external functions saves gas                                         |    28     |
| [G-006] | Multiple accesses of a `mapping/array` should use a local variable cache                                                           |     1     |
| [G-007] | internal functions only called once can be inlined to save gas                                                                     |     3     |
| [G-008] | Replace modifier with function                                                                                                     |     4     |

## [G-001] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact

Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings

Total:5

[src/Drips.sol#L284](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L284)

```solidity
284:    toCycle -= receivableCycles;
```

[src/Drips.sol#L289](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L289)

```solidity
289:    receivedAmt += uint128(amtPerCycle);
```

[src/Splits.sol#L235](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L235)

```solidity
235:    totalWeight += weight;
```

[src/Splits.sol#L167](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L167)

```solidity
167:    collectableAmt -= splitAmt;
```

[src/Splits.sol#L162](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L162)

```solidity
162:    splitAmt += currSplitAmt;
```

## [G-002] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:1

[src/Drips.sol#L355](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L355)

```solidity
355:    bytes32[] memory squeezedHistoryHashes = new bytes32[](squeezedNum);
```

### Recommendation

Using `storage` instead of `memory`

## [G-003] Functions guaranteed to revert when called by normal users can be marked payable

### Impact

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

Total:5

[src/Managed.sol#L122](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L122)

```solidity
122:    function pause() public onlyAdminOrPauser whenNotPaused {
```

[src/Managed.sol#L97](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L97)

```solidity
97:    function revokePauser(address pauser) public onlyAdmin {
```

[src/Managed.sol#L128](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L128)

```solidity
128:    function unpause() public onlyAdminOrPauser whenPaused {
```

[src/Managed.sol#L90](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L90)

```solidity
90:    function grantPauser(address pauser) public onlyAdmin {
```

[src/Managed.sol#L84](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L84)

```solidity
84:    function changeAdmin(address newAdmin) public onlyAdmin {
```

### Recommendation

Mark the function as payable.

## [G-004] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Impact

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

### Findings

Total:2

[src/DripsHub.sol#L99-L101](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L99-L101)

```solidity
99:    mapping(uint32 => address) driverAddresses;
100:            /// @notice The total amount currently stored in DripsHub of each token.
101:            mapping(IERC20 => uint256) totalBalances;
```

[src/Caller.sol#L88-L90](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L88-L90)

```solidity
88:    mapping(address => EnumerableSet.AddressSet) internal _authorized;
89:        /// @notice The nonce which needs to be used in the next EIP-712 message signed by the address.
90:        mapping(address => uint256) public nonce;
```

### Recommendation

Same as description

## [G-005] Using `calldata` instead of memory for read-only arguments in external functions saves gas

### Impact

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a `for-loop` to copy each index of the `calldata` to the `memory` index. Each iteration of this `for-loop` costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead.<br><br> If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

### Findings

Total:28

[src/Drips.sol#L834](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L834)

```solidity
834:    function _hashDrips(DripsReceiver[] memory receivers)
```

[src/Drips.sol#L792](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L792)

```solidity
792:    function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver)
```

[src/Drips.sol#L815](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L815)

```solidity
815:    function _getConfig(uint256[] memory configs, uint256 idx)
```

[src/Drips.sol#L769](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L769)

```solidity
769:    function _buildConfigs(DripsReceiver[] memory receivers)
```

[src/DripsHub.sol#L270-L273](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L270-L273)

```solidity
270:    function balanceAt(
271:            uint256 userId,
272:            IERC20 erc20,
273:            DripsReceiver[] memory receivers,
```

[src/DripsHub.sol#L297-L300](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L297-L300)

```solidity
297:    function balanceAt(
298:            uint256 userId,
299:            IERC20 erc20,
300:            DripsReceiver[] memory receivers,
```

[src/DripsHub.sol#L460-L463](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L460-L463)

```solidity
460:    function balanceAt(
461:            uint256 userId,
462:            IERC20 erc20,
463:            DripsReceiver[] memory receivers,
```

[src/DripsHub.sol#L510-L513](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L510-L513)

```solidity
510:    function balanceAt(
511:            uint256 userId,
512:            IERC20 erc20,
513:            DripsReceiver[] memory receivers,
```

[src/DripsHub.sol#L355](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L355)

```solidity
355:    function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)
```

[src/DripsHub.sol#L328](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L328)

```solidity
328:    function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
```

[src/DripsHub.sol#L576](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L576)

```solidity
576:    function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
```

[src/DripsHub.sol#L270-L275](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L270-L275)

```solidity
270:    function setDrips(
271:            uint256 userId,
272:            IERC20 erc20,
273:            DripsReceiver[] memory currReceivers,
274:            int128 balanceDelta,
275:            DripsReceiver[] memory newReceivers,
```

[src/DripsHub.sol#L297-L302](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L297-L302)

```solidity
297:    function setDrips(
298:            uint256 userId,
299:            IERC20 erc20,
300:            DripsReceiver[] memory currReceivers,
301:            int128 balanceDelta,
302:            DripsReceiver[] memory newReceivers,
```

[src/DripsHub.sol#L460-L465](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L460-L465)

```solidity
460:    function setDrips(
461:            uint256 userId,
462:            IERC20 erc20,
463:            DripsReceiver[] memory currReceivers,
464:            int128 balanceDelta,
465:            DripsReceiver[] memory newReceivers,
```

[src/DripsHub.sol#L510-L515](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L510-L515)

```solidity
510:    function setDrips(
511:            uint256 userId,
512:            IERC20 erc20,
513:            DripsReceiver[] memory currReceivers,
514:            int128 balanceDelta,
515:            DripsReceiver[] memory newReceivers,
```

[src/DripsHub.sol#L270-L275](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L270-L275)

```solidity
270:    function squeezeDrips(
271:            uint256 userId,
272:            IERC20 erc20,
273:            uint256 senderId,
274:            bytes32 historyHash,
275:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L297-L302](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L297-L302)

```solidity
297:    function squeezeDrips(
298:            uint256 userId,
299:            IERC20 erc20,
300:            uint256 senderId,
301:            bytes32 historyHash,
302:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L460-L465](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L460-L465)

```solidity
460:    function squeezeDrips(
461:            uint256 userId,
462:            IERC20 erc20,
463:            uint256 senderId,
464:            bytes32 historyHash,
465:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L510-L515](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L510-L515)

```solidity
510:    function squeezeDrips(
511:            uint256 userId,
512:            IERC20 erc20,
513:            uint256 senderId,
514:            bytes32 historyHash,
515:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L270-L275](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L270-L275)

```solidity
270:    function squeezeDripsResult(
271:            uint256 userId,
272:            IERC20 erc20,
273:            uint256 senderId,
274:            bytes32 historyHash,
275:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L297-L302](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L297-L302)

```solidity
297:    function squeezeDripsResult(
298:            uint256 userId,
299:            IERC20 erc20,
300:            uint256 senderId,
301:            bytes32 historyHash,
302:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L460-L465](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L460-L465)

```solidity
460:    function squeezeDripsResult(
461:            uint256 userId,
462:            IERC20 erc20,
463:            uint256 senderId,
464:            bytes32 historyHash,
465:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L510-L515](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L510-L515)

```solidity
510:    function squeezeDripsResult(
511:            uint256 userId,
512:            IERC20 erc20,
513:            uint256 senderId,
514:            bytes32 historyHash,
515:            DripsHistory[] memory dripsHistory
```

[src/DripsHub.sol#L595](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L595)

```solidity
595:    function hashSplits(SplitsReceiver[] memory receivers)
```

[src/Splits.sol#L143](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L143)

```solidity
143:    function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)
```

[src/Splits.sol#L117](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L117)

```solidity
117:    function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
```

[src/Splits.sol#L270](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L270)

```solidity
270:    function _hashSplits(SplitsReceiver[] memory receivers)
```

[src/Splits.sol#L250](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L250)

```solidity
250:    function _assertCurrSplits(uint256 userId, SplitsReceiver[] memory currReceivers)
```

### Recommendation

Same as description

## [G-006] Multiple accesses of a `mapping/array` should use a local variable cache

### Impact

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into `memory/calldata` 

### Findings

Total:1

[src/Drips.sol#L363](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L363)

```solidity
363:    nextSqueezed[currCycleConfigs - revIdx] = _currTimestamp();
```

### Recommendation

Same as description

## [G-007] internal functions only called once can be inlined to save gas

### Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

### Findings

Total:3

[src/Splits.sol#L262](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L262)

```solidity
262:    function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {
```

[src/NFTDriver.sol#L300](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L300)

```solidity
300:    function _msgData() internal view override(Context, ERC2771Context) returns (bytes calldata) {
```

[src/Managed.sol#L136](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L136)

```solidity
136:    function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
```

### Recommendation

Same as description

## [G-008] Replace modifier with function

### Impact

Modifiers make code more elegant, but cost more than normal functions.

### Findings

Total:4

[src/Managed.sol#L52](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L52)

```solidity
52:    modifier onlyAdminOrPauser() {
```

[src/Managed.sol#L60](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L60)

```solidity
60:    modifier whenNotPaused() {
```

[src/Managed.sol#L66](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L66)

```solidity
66:    modifier whenPaused() {
```

[src/Managed.sol#L46](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L46)

```solidity
46:    modifier onlyAdmin() {
```
