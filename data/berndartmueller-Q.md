# QA Report

## Table of Contents

- [Low-Risk Findings](#low-risk-findings)
  - [\[L-01\] An authorized user can unauthorize other authorized users of the same sender](#l-01-an-authorized-user-can-unauthorize-other-authorized-users-of-the-same-sender)
  - [\[L-02\] Lack of reasonable boundaries for cycle secs](#l-02-lack-of-reasonable-boundaries-for-cycle-secs)
  - [\[L-03\] Loading a drip configuration with assembly in `_getConfig` is unsafe](#l-03-loading-a-drip-configuration-with-assembly-in-_getconfig-is-unsafe)
  - [\[L-04\] Unused `dripId` of receiver config is used considered while sorting receivers](#l-04-unused-dripid-of-receiver-config-is-used-considered-while-sorting-receivers)
  - [\[L-05\] Collecting funds should be usable while the `DripsHub` contract is paused](#l-05-collecting-funds-should-be-usable-while-the-dripshub-contract-is-paused)
  - [\[L-06\] An immutable split is unable to collect funds if it has itself set as a split receiver](#l-06-an-immutable-split-is-unable-to-collect-funds-if-it-has-itself-set-as-a-split-receiver)
  - [\[L-07\] Precision issues caused by division before multiplication in `Drips._drippedAmt`](#l-07-precision-issues-caused-by-division-before-multiplication-in-drips_drippedamt)
- [Non-Critical Findings](#non-critical-findings)
  - [\[NC-01\] The NatSpec comment of the `DripsConfigImpl.lt` function neglects to sorting of `dripId`](#nc-01-the-natspec-comment-of-the-dripsconfigimpllt-function-neglects-to-sorting-of-dripid)
  - [\[NC-02\] Unused `amtPerCycle` variable in `Drips._drippedAmt` calculation](#nc-02-unused-amtpercycle-variable-in-drips_drippedamt-calculation)

## Low-Risk Findings

### [L-01] An authorized user can unauthorize other authorized users of the same sender

A user can grant authorization to another address to make calls on their behalf via the `Caller.callAs` function. The `Caller.unauthorize` function allows the user to revoke the authorization of another address.

An authorized user can revoke the authorization of another authorized user of the same sender. This is because the authorized user can call the `Caller.unauthorize` function on behalf of the sender.

#### Findings

[Caller.sol#L114-L118](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L114-L118)

```solidity
114: function unauthorize(address user) public {
115:     address sender = _msgSender();
116:     require(_authorized[sender].remove(user), "Address is not authorized");
117:     emit Unauthorized(sender, user);
118: }
```

#### Recommended mitigation steps

Consider preventing calls to the `Caller` contract address from within the `_call` function

### [L-02] Lack of reasonable boundaries for cycle secs

If `_cycleSecs` is set too low, for example, to a value less than the Ethereum block time of 12 sec, it will not be possible to squeeze. If set too high, e.g., larger than the current `block.timestamp`, it will not be possible to receive drips - only squeezing will be possible.

#### Findings

[Drips.sol#L221](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L221)

```solidity
219: constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
220:     require(cycleSecs > 1, "Cycle length too low");
221:     _cycleSecs = cycleSecs;
222:     _dripsStorageSlot = dripsStorageSlot;
223: }
```

#### Recommended mitigation steps

Consider adding an appropriate upper limit for `_cycleSecs`.

### [L-03] Loading a drip configuration with assembly in `_getConfig` is unsafe

The `Drips._getConfig` loads a drips configuration and returns the `amtPerSec`, `start` and `end` values. The `configs` parameter is a `uint256[]` array, containing the drips config.

As seen in `Drips.create`, the drip config has the `dripId` in the 32 most significant bits, followed by the `amtPerSec` in the next 160 bits.

Even though the `dripId` is not used in the contract and it's even stripped out ([see Drips.sol#L805](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L805)) when preprocessing the drips receiver in `Drips._addConfig`, it could cause serious trouble in the future if the `dripId` is used in the contract and not taken into account when loading with `Drips._getConfig`.

#### Findings

[Drips.sol#L825](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L825)

```solidity
815: function _getConfig(uint256[] memory configs, uint256 idx)
816:     private
817:     pure
818:     returns (uint256 amtPerSec, uint256 start, uint256 end)
819: {
820:     uint256 val;
821:     // slither-disable-next-line assembly
822:     assembly ("memory-safe") {
823:         val := mload(add(32, add(configs, shl(5, idx))))
824:     }
825:     return (val >> 64, uint32(val >> 32), uint32(val));
826: }
```

#### Recommended mitigation steps

Consider truncating the `uint256` value to `uint160` before returning it.

### [L-04] Unused `dripId` of receiver config is used considered while sorting receivers

The `Drips._isOrdered` function compares two given receivers. The receiver config can include the prefixed `dripId` in the 32 most significant bits. This allows using the `dripId` for changing the order of receivers when setting a new drips configuration without violating the sorting requirement, _First compares their `amtPerSec`s, then their `start`s and then their `duration`s_ ([see Drips.sol#L93](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L93)).

While no harmful effects are expected, it can be used in the `Drips._setDrips` function to control if delta amounts of receivers are first removed, added, or updated, are expected.

#### Findings

[Drips.sol#L1069](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1069)

```solidity
1061: function _isOrdered(DripsReceiver memory prev, DripsReceiver memory next)
1062:     private
1063:     pure
1064:     returns (bool)
1065: {
1066:     if (prev.userId != next.userId) {
1067:         return prev.userId < next.userId;
1068:     }
1069:     return prev.config.lt(next.config);
1070: }
```

#### Recommended mitigation steps

Consider omitting the `dripId` from the receiver config when comparing the receivers.

### [L-05] Collecting funds should be usable while the `DripsHub` contract is paused

The `DripsHub.collect` function is only callable when the `DripsHub` contract is not paused. This prevents a user from collecting accumulated funds. The admin of the `DripsHub` contract can potentially pause the contracts at any time, locking users out of their honestly earned funds.

#### Findings

[DripsHub.sol#L388](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L388)

```solidity
386: function collect(uint256 userId, IERC20 erc20)
387:     public
388:     whenNotPaused
389:     onlyDriver(userId)
390:     returns (uint128 amt)
391: {
392:     amt = Splits._collect(userId, _assetId(erc20));
393:     _decreaseTotalBalance(erc20, amt);
394:     erc20.safeTransfer(msg.sender, amt);
395: }
```

#### Recommended mitigation steps

Consider removing the `whenNotPaused` modifier to allow collecting funds while the `DripsHub` contract is paused.

### [L-06] An immutable split is unable to collect funds if it has itself set as a split receiver

An immutable split cannot collect funds if it has itself set as a split receiver. This is because the `ImmutableSplitsDriver` contract lacks the functionality to collect available funds.

#### Findings

[ImmutableSplitsDriver.sol#L66](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L66)

```solidity
53: function createSplits(SplitsReceiver[] calldata receivers, UserMetadata[] calldata userMetadata)
54:     public
55:     whenNotPaused
56:     returns (uint256 userId)
57: {
58:     userId = nextUserId();
59:     StorageSlot.getUint256Slot(_counterSlot).value++;
60:     uint256 weightSum = 0;
61:     for (uint256 i = 0; i < receivers.length; i++) {
62:         weightSum += receivers[i].weight;
63:     }
64:     require(weightSum == totalSplitsWeight, "Invalid total receivers weight");
65:     emit CreatedSplits(userId, dripsHub.hashSplits(receivers));
66:     dripsHub.setSplits(userId, receivers);
67:     if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);
68: }
```

#### Recommended mitigation steps

Consider preventing setting one of the receivers to the user ID of the newly created immutable split.

### [L-07] Precision issues caused by division before multiplication in `Drips._drippedAmt`

The `Drips._drippedAmt` function calculates the amount of dripped funds for a given time interval and `amtPerSec`.

However, by dividing the result of `mul(cycleSecs, amtPerSec)` by `_AMT_PER_SEC_MULTIPLIER` before multiplying it by `endedCycles`, the precision of the result is reduced. This can cause the amount of dripped funds to be lower if the multiplication would have been performed first.

#### Findings

[Drips.sol#L1107](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1107)

```solidity
1093: function _drippedAmt(uint256 amtPerSec, uint256 start, uint256 end)
1094:     private
1095:     view
1096:     returns (uint256 amt)
1097: {
1098:     // This function is written in Yul because it can be called thousands of times
1099:     // per transaction and it needs to be optimized as much as possible.
1100:     // As of Solidity 0.8.13, rewriting it in unchecked Solidity triples its gas cost.
1101:     uint256 cycleSecs = _cycleSecs;
1102:     // slither-disable-next-line assembly
1103:     assembly {
1104:         let endedCycles := sub(div(end, cycleSecs), div(start, cycleSecs))
1105:         // slither-disable-next-line divide-before-multiply
1106:         let amtPerCycle := div(mul(cycleSecs, amtPerSec), _AMT_PER_SEC_MULTIPLIER) line
1107:         amt := mul(endedCycles, div(mul(cycleSecs, amtPerSec), _AMT_PER_SEC_MULTIPLIER)) // @audit-info Division before before multiplication
1108:         // slither-disable-next-line weak-prng
1109:         let amtEnd := div(mul(mod(end, cycleSecs), amtPerSec), _AMT_PER_SEC_MULTIPLIER)
1110:         amt := add(amt, amtEnd)
1111:         // slither-disable-next-line weak-prng
1112:         let amtStart := div(mul(mod(start, cycleSecs), amtPerSec), _AMT_PER_SEC_MULTIPLIER)
1113:         amt := sub(amt, amtStart)
1114:     }
1115: }
```

#### Recommended mitigation steps

Consider multiplying `mul(cycleSecs, amtPerSec)` by `endedCycles` before dividing by `_AMT_PER_SEC_MULTIPLIER` in line 1107 to avoid precision issues.

## Non-Critical Findings

### [NC-01] The NatSpec comment of the `DripsConfigImpl.lt` function neglects to sorting of `dripId`

According to the NatSpec comment of the `DripsConfigImpl.lt` function, the `DripsConfig` struct is sorted by `amtPerSec`, then `start` and then `duration`. However, the `dripId` is not mentioned in the comment:

> /// First compares their `amtPerSec`s, then their `start`s and then their `duration`s.

#### Findings

[Drips.sol#L93](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L93)

```solidity
92: /// @notice Compares two `DripsConfig`s.
93: /// First compares their `amtPerSec`s, then their `start`s and then their `duration`s.
94: function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {
95:     return DripsConfig.unwrap(config) < DripsConfig.unwrap(otherConfig);
96: }
```

[Drips.sol#L1069](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1069)

```solidity
1061: function _isOrdered(DripsReceiver memory prev, DripsReceiver memory next)
1062:     private
1063:     pure
1064:     returns (bool)
1065: {
1066:     if (prev.userId != next.userId) {
1067:         return prev.userId < next.userId;
1068:     }
1069:     return prev.config.lt(next.config); // @audit-info `dripId` of receiver config (The 32 most significant bits) is considered while sorting receivers
1070: }
```

#### Recommended mitigation steps

Consider updating the NatSpec comment to include the `dripId` in the sorting order.

### [NC-02] Unused `amtPerCycle` variable in `Drips._drippedAmt` calculation

The `Drips._drippedAmt` function calculates the dripped amount. The `amtPerCycle` variable is not used in the calculation and can be removed.

#### Findings

[Drips.sol#L1106](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1106)

```solidity
1093: function _drippedAmt(uint256 amtPerSec, uint256 start, uint256 end)
1094:     private
1095:     view
1096:     returns (uint256 amt)
1097: {
1098:     // This function is written in Yul because it can be called thousands of times
1099:     // per transaction and it needs to be optimized as much as possible.
1100:     // As of Solidity 0.8.13, rewriting it in unchecked Solidity triples its gas cost.
1101:     uint256 cycleSecs = _cycleSecs;
1102:     // slither-disable-next-line assembly
1103:     assembly {
1104:         let endedCycles := sub(div(end, cycleSecs), div(start, cycleSecs))
1105:         // slither-disable-next-line divide-before-multiply
1106:         let amtPerCycle := div(mul(cycleSecs, amtPerSec), _AMT_PER_SEC_MULTIPLIER) // @audit-info Unused variable
1107:         amt := mul(endedCycles, div(mul(cycleSecs, amtPerSec), _AMT_PER_SEC_MULTIPLIER))
1108:         // slither-disable-next-line weak-prng
1109:         let amtEnd := div(mul(mod(end, cycleSecs), amtPerSec), _AMT_PER_SEC_MULTIPLIER)
1110:         amt := add(amt, amtEnd)
1111:         // slither-disable-next-line weak-prng
1112:         let amtStart := div(mul(mod(start, cycleSecs), amtPerSec), _AMT_PER_SEC_MULTIPLIER)
1113:         amt := sub(amt, amtStart)
1114:     }
1115: }
```

#### Recommended mitigation steps

Consider removing the unused variable.
