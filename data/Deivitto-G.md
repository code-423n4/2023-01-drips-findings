
# Gas

| | Issue index |
| ----------- | ----------- |
| 1 | [Multiple `address`/`ID` `mappings` can be combined into a single `mapping` of an `address`/`ID` to a `struct`, where appropriate](#multiple-`address`/`id`-`mappings`-can-be-combined-into-a-single-`mapping`-of-an-`address`/`id`-to-a-`struct`,-where-appropriate) |
| 2 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#`<x>-+=-<y>`-costs-more-gas-than-`<x>-=-<x>-+-<y>`-for-state-variables) |
| 3 | [`++i / i++` should be `unchecked{++i} / unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#`++i-/-i++`-should-be-`unchecked{++i}-/-unchecked{i++}`-when-it-is-not-possible-for-them-to-overflow,-as-is-the-case-when-used-in-for-loop-and-while-loops) |
| 4 | [`require()`/`revert()` strings longer than 32 bytes cost extra gas](#`require()`/`revert()`-strings-longer-than-32-bytes-cost-extra-gas) |
| 5 | [`internal` functions only called once can be inlined to save gas](#`internal`-functions-only-called-once-can-be-inlined-to-save-gas) |
| 6 | [`bytes` constants are more efficient than `string` constants](#`bytes`-constants-are-more-efficient-than-`string`-constants) |
| 7 | [`abi.encode()` is less gas efficient than `abi.encodePacked()`](#`abi.encode()`-is-less-gas-efficient-than-`abi.encodepacked()`) |
| 8 | [Duplicated `require()` check should be refactored](#duplicated-`require()`-check-should-be-refactored) |

## Multiple `address`/`ID` `mappings` can be combined into a single `mapping` of an `address`/`ID` to a `struct`, where appropriate

if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s `keccak256` hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

- [Caller.sol#L88](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L88)
    mapping(address => EnumerableSet.AddressSet) internal _authorized;

- [Caller.sol#L90](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L90)
    mapping(address => uint256) public nonce;

## `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

Using the addition operator instead of plus-equals saves gas. Same goes for other operands like `-=`.

- [Drips.sol#L253](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L253)
                amtDeltas[toCycle].thisCycle += finalAmtPerCycle;

- [Drips.sol#L1053](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1053)
            amtDelta.thisCycle += int128(fullCycle - nextCycle);

- [Drips.sol#L1054](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1054)
            amtDelta.nextCycle += int128(nextCycle);

- [DripsHub.sol#L632](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L632)
        totalBalances[erc20] += amt;

- [Splits.sol#L168](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L168)
        balance.collectable += collectableAmt;

## `++i / i++` should be `unchecked{++i} / unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

- [Drips.sol#L247](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L247)
            for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

- [Drips.sol#L287](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L287)
        for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

- [Drips.sol#L358](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L358)
        for (uint256 i = 0; i < squeezedNum; i++) {

- [Drips.sol#L422](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L422)
        for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

- [Drips.sol#L450](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L450)
        for (uint256 i = 0; i < dripsHistory.length; i++) {

- [Drips.sol#L490](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L490)
        for (; idx < receivers.length; idx++) {

- [Drips.sol#L563](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L563)
        for (uint256 i = 0; i < receivers.length; i++) {

- [Drips.sol#L664](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L664)
            for (uint256 i = 0; i < newReceivers.length; i++) {

- [Drips.sol#L745](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L745)
            for (uint256 i = 0; i < configsLen; i++) {

- [Drips.sol#L777](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L777)
            for (uint256 i = 0; i < receivers.length; i++) {

- [DripsHub.sol#L613](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L613)
        for (uint256 i = 0; i < userMetadata.length; i++) {

- [Splits.sol#L127](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L127)
        for (uint256 i = 0; i < currReceivers.length; i++) {

- [Splits.sol#L158](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L158)
        for (uint256 i = 0; i < currReceivers.length; i++) {

- [Splits.sol#L231](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L231)
        for (uint256 i = 0; i < receivers.length; i++) {

- [Caller.sol#L196](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L196)
        for (uint256 i = 0; i < calls.length; i++) {

- [ImmutableSplitsDriver.sol#L61](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L61)
        for (uint256 i = 0; i < receivers.length; i++) {

## `require()`/`revert()` strings longer than 32 bytes cost extra gas

`NOTE`: None of these findings where found by [4naly3er output - Gas](https://gist.github.com/supernovahs/ed0ecfe251e3adf824bb669be981bca9)

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas

- [NFTDriver.sol#L43](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L43)

- [Managed.sol#L54](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L54)


## `internal` functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

`duration`

- [Drips.sol#L88](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L88)
    function duration(DripsConfig config) internal pure returns (uint32) {

## `bytes` constants are more efficient than `string` constants

If data can fit into 32 bytes, then you should use `bytes32` datatype rather than bytes or strings as it is cheaper in solidity.

- [Caller.sol#L82](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L82)
    string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("


## `abi.encode()` is less gas efficient than `abi.encodePacked()`

Changing the abi.encode function to `abi.encodePacked` can save gas since the `abi.encode` function pads extra null `bytes` at the end of the call data, which is unnecessary. Also, in general, `abi.encodePacked` is more gas-efficient.

- [Drips.sol#L842](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L842)
        return keccak256(abi.encode(receivers));

- [Drips.sol#L858](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L858)
        return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));

- [Splits.sol#L278](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L278)
        return keccak256(abi.encode(receivers));

- [Caller.sol#L176](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L176)
            abi.encode(

## Duplicated `require()` check should be refactored

Duplicated `require()` / `revert()` checks should be refactored to a modifier or function to save gas. If it's been used for example `require(addr != address(0))` or `require(a > b)`, etc multiple times in the same contract. This can be refactored to a modifier like `notZeroAddress(address a)`, function `biggerThan(uint a, uint b)`, etc.

- [Drips.sol#L541](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L541)
        require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list");

- [Drips.sol#L622](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L622)
        require(_hashDrips(currReceivers) == state.dripsHash, "Invalid current drips list");


- [Caller.sol#L116](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L116)
        require(_authorized[sender].remove(user), "Address is not authorized");

- [Caller.sol#L149](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L149)
        require(isAuthorized(sender, _msgSender()), "Not authorized");


