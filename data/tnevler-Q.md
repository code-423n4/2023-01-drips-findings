# Report
## Non-Critical Issues ##
### [N-1]: Function defines a named return variable but then it uses return statements
**Context:**

1. ```return toCycle - fromCycle;``` [L306](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L306) 
1. ```return uint128(amt);``` [L497](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L497) 
1. ```return``` [L520](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L520) 
1. ```return _balanceAt(state.balance, state.updateTime, state.maxEnd, receivers, timestamp);``` [L542](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L542) 
1. ```return false;``` [L757](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L757) 
1. ```return true;``` [L760](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L760) 
1. ```return configsLen + 1;``` [L806](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L806) 
1. ```return (val >> 64, uint32(val >> 32), uint32(val));``` [L825](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L825) 
1. ```return bytes32(0);``` [L840](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L840) 
1. ```return keccak256(abi.encode(receivers));``` [L842](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L842) 
1. ```return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));``` [L858](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L858) 
1. ```return _dripsRange(receiver, updateTime, maxEnd, _currTimestamp(), type(uint32).max);``` [L975](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L975) 
1. ```return (start, uint32(end));``` [L1012](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1012) 
1. ```return timestamp / _cycleSecs + 1;``` [L1122](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1122) 
1. ```return uint32(block.timestamp);``` [L1129](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1129) 
1. ```return currTimestamp - (currTimestamp % _cycleSecs);``` [L1137](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1137) 
1. ```return _dripsHubStorage().driverAddresses[driverId];``` [L146](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L146) 
1. ```return _dripsHubStorage().nextDriverId;``` [L161](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L161) 
1. ```return Drips._cycleSecs;``` [L170](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L170) 
1. ```return _dripsHubStorage().totalBalances[erc20];``` [L182](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L182) 
1. ```return Drips._receivableDripsCycles(userId, _assetId(erc20));``` [L201](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L201) 
1. ```return Splits._splittable(userId, _assetId(erc20));``` [L318](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L318) 
1. ```return Splits._splitResult(userId, currReceivers, amount);``` [L333](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L333) 
1. ```return Splits._split(userId, _assetId(erc20), currReceivers);``` [L360](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L360) 
1. ```return Splits._collectable(userId, _assetId(erc20));``` [L373](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L373) 
1. ```return Drips._dripsState(userId, _assetId(erc20));``` [L443](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L443) 
1. ```return Drips._balanceAt(userId, _assetId(erc20), receivers, timestamp);``` [L466](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L466) 
1. ```return Drips._hashDrips(receivers);``` [L547](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L547) 
1. ```return Drips._hashDripsHistory(oldDripsHistoryHash, dripsHash, updateTime, maxEnd);``` [L563](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L563) 
1. ```return Splits._splitsHash(userId);``` [L588](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L588) 
1. ```return Splits._hashSplits(receivers);``` [L600](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L600) 
1. ```return uint160(address(erc20));``` [L643](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L643) 
1. ```return _splitsStorage().splitsStates[userId].balances[assetId].splittable;``` [L107](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L107) 
1. ```return (0, 0);``` [L153](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L153) 
1. ```return _splitsStorage().splitsStates[userId].balances[assetId].collectable;``` [L177](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L177) 
1. ```return _splitsStorage().splitsStates[userId].splitsHash;``` [L263](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L263) 
1. ```return bytes32(0);``` [L276](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L276) 
1. ```return keccak256(abi.encode(receivers));``` [L278](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L278) 
1. ```return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_mintedTokensSlot).value;``` [L51](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L51) 
1. ```return _managedStorage().pausers.contains(pauser);``` [L106](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L106) 
1. ```return _managedStorage().pausers.values();``` [L113](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L113) 
1. ```return _managedStorage().isPaused;``` [L118](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L118) 
1. ```return bytes32(uint256(keccak256(bytes(name))) - 1);``` [L137](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L137) 
1. ```return _authorized[sender].contains(user);``` [L125](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L125) 
1. ```return _authorized[sender].values();``` [L133](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L133) 
1. ```return _call(sender, to, data, msg.value);``` [L150](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L150) 
1. ```return _call(sender, to, data, msg.value);``` [L182](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L182) 
1. ```return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);``` [L207](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L207) 
1. ```return (uint256(driverId) << 224) | uint160(userAddr);``` [L41](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L41) 
1. ```return calcUserId(_msgSender());``` [L47](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L47) 
1. ```return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_counterSlot).value;``` [L37](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L37) 

**Recommendation:**

Choose named return variable or return statement. It is unnecessary to use both.

### [N-2]: Use of immutable instead of constant keccak expression
**Context:**

```bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));``` [L84](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L84) 

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/contracts.html#constant-and-immutable-state-variables) for a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. It is recommended to use immutable instead. 

### [N-3]: Wrong order of functions
**Context:**

1. ```struct SplitsStorage {``` [L73](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L73) (struct definition can not go after event definition)
1. ```modifier onlyHolder(uint256 tokenId) {``` [L40](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L40) (modifier definition can not go after constructor)
1. ```function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {``` [L60](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L60) (public function can not go after internal function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-4]: NatSpec is missing
**Context:**

1. ```function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {``` [L629](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L629) 
1. ```function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {``` [L635](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L635) 
1. ```function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal {``` [L98](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L98) 
1. ```function _transferFromCaller(IERC20 erc20, uint128 amt) internal {``` [L285](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L285) 
1. ```function _call(address sender, address to, bytes memory data, uint256 value)``` [L202](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L202) 