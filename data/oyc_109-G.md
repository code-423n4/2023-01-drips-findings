## [G-01] Don't Initialize Variables with Default Value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```
2023-01-drips/src/Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {
2023-01-drips/src/Drips.sol::358 => for (uint256 i = 0; i < squeezedNum; i++) {
2023-01-drips/src/Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips/src/Drips.sol::478 => uint256 idx = 0;
2023-01-drips/src/Drips.sol::489 => uint256 amt = 0;
2023-01-drips/src/Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips/src/Drips.sol::744 => uint256 spent = 0;
2023-01-drips/src/Drips.sol::745 => for (uint256 i = 0; i < configsLen; i++) {
2023-01-drips/src/Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Drips.sol::880 => uint256 currIdx = 0;
2023-01-drips/src/Drips.sol::881 => uint256 newIdx = 0;
2023-01-drips/src/DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {
2023-01-drips/src/ImmutableSplitsDriver.sol::60 => uint256 weightSum = 0;
2023-01-drips/src/ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Splits.sol::126 => uint32 splitsWeight = 0;
2023-01-drips/src/Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::157 => uint32 splitsWeight = 0;
2023-01-drips/src/Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::228 => uint64 totalWeight = 0;
2023-01-drips/src/Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
```

## [G-02] Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

```
2023-01-drips/src/Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {
2023-01-drips/src/Drips.sol::422 => for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
2023-01-drips/src/Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips/src/Drips.sol::479 => for (uint256 idxCap = receivers.length; idx < idxCap;) {
2023-01-drips/src/Drips.sol::490 => for (; idx < receivers.length; idx++) {
2023-01-drips/src/Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips/src/Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {
2023-01-drips/src/ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
```

## [G-03] Long Revert Strings

Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

```
2023-01-drips/src/Drips.sol::454 => require(dripsHash == 0, "Drips history entry with hash and receivers");
2023-01-drips/src/Drips.sol::540 => require(timestamp >= state.updateTime, "Timestamp before last drips update");
2023-01-drips/src/Splits.sol::239 => require(prevUserId < userId, "Splits receivers not sorted by user ID");
```

## [G-04] Use Shift Right/Left instead of Division/Multiplication if possible

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

```
2023-01-drips/src/Drips.sol::480 => uint256 idxMid = (idx + idxCap) / 2;
2023-01-drips/src/Drips.sol::717 => uint256 end = (enoughEnd + notEnoughEnd) / 2;
```

## [G-05] Use calldata instead of memory

Use calldata instead of memory for function parameters saves gas if the function argument is only read.

```
2023-01-drips/src/Caller.sol::193 => function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {
2023-01-drips/src/DripsHub.sol::546 => function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {
2023-01-drips/src/Managed.sol::136 => function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
```

## [G-06] Using private rather than public for constants, saves gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```
2023-01-drips/src/AddressDriver.sol::23 => DripsHub public immutable dripsHub;
2023-01-drips/src/AddressDriver.sol::25 => uint32 public immutable driverId;
2023-01-drips/src/ImmutableSplitsDriver.sol::13 => DripsHub public immutable dripsHub;
2023-01-drips/src/ImmutableSplitsDriver.sol::15 => uint32 public immutable driverId;
2023-01-drips/src/ImmutableSplitsDriver.sol::17 => uint32 public immutable totalSplitsWeight;
2023-01-drips/src/NFTDriver.sol::23 => DripsHub public immutable dripsHub;
2023-01-drips/src/NFTDriver.sol::25 => uint32 public immutable driverId;
```

## [G-07] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```
2023-01-drips/src/Managed.sol::84 => function changeAdmin(address newAdmin) public onlyAdmin {
2023-01-drips/src/Managed.sol::90 => function grantPauser(address pauser) public onlyAdmin {
2023-01-drips/src/Managed.sol::97 => function revokePauser(address pauser) public onlyAdmin {
2023-01-drips/src/Managed.sol::122 => function pause() public onlyAdminOrPauser whenNotPaused {
2023-01-drips/src/Managed.sol::128 => function unpause() public onlyAdminOrPauser whenPaused {
```

## [G-08] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```
2023-01-drips/src/AddressDriver.sol::25 => uint32 public immutable driverId;
2023-01-drips/src/Drips.sol::28 => uint32 updateTime;
2023-01-drips/src/Drips.sol::30 => uint32 maxEnd;
2023-01-drips/src/Drips.sol::108 => uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
2023-01-drips/src/Drips.sol::112 => uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
2023-01-drips/src/Drips.sol::117 => uint32 internal immutable _cycleSecs;
2023-01-drips/src/Drips.sol::189 => uint32 nextReceivableCycle;
2023-01-drips/src/Drips.sol::191 => uint32 updateTime;
2023-01-drips/src/Drips.sol::193 => uint32 maxEnd;
2023-01-drips/src/Drips.sol::195 => uint128 balance;
2023-01-drips/src/Drips.sol::197 => uint32 currCycleConfigs;
2023-01-drips/src/Drips.sol::208 => int128 thisCycle;
2023-01-drips/src/Drips.sol::210 => int128 nextCycle;
2023-01-drips/src/Drips.sol::237 => uint32 receivableCycles;
2023-01-drips/src/Drips.sol::238 => uint32 fromCycle;
2023-01-drips/src/Drips.sol::239 => uint32 toCycle;
2023-01-drips/src/Drips.sol::240 => int128 finalAmtPerCycle;
2023-01-drips/src/Drips.sol::357 => uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
2023-01-drips/src/Drips.sol::365 => uint32 cycleStart = _currCycleStart();
2023-01-drips/src/Drips.sol::421 => uint32 squeezeEndCap = _currTimestamp();
2023-01-drips/src/Drips.sol::487 => uint32 updateTime = dripsHistory.updateTime;
2023-01-drips/src/Drips.sol::488 => uint32 maxEnd = dripsHistory.maxEnd;
2023-01-drips/src/Drips.sol::623 => uint32 lastUpdate = state.updateTime;
2023-01-drips/src/Drips.sol::624 => uint128 newBalance;
2023-01-drips/src/Drips.sol::625 => uint32 newMaxEnd;
2023-01-drips/src/Drips.sol::1135 => uint32 currTimestamp = _currTimestamp();
2023-01-drips/src/DripsHub.sol::58 => uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
2023-01-drips/src/DripsHub.sol::64 => uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
2023-01-drips/src/DripsHub.sol::97 => uint32 nextDriverId;
2023-01-drips/src/DripsHub.sol::119 => uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
2023-01-drips/src/ImmutableSplitsDriver.sol::15 => uint32 public immutable driverId;
2023-01-drips/src/ImmutableSplitsDriver.sol::17 => uint32 public immutable totalSplitsWeight;
2023-01-drips/src/NFTDriver.sol::25 => uint32 public immutable driverId;
2023-01-drips/src/Splits.sol::11 => uint32 weight;
2023-01-drips/src/Splits.sol::22 => uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
2023-01-drips/src/Splits.sol::25 => uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;
2023-01-drips/src/Splits.sol::88 => uint128 splittable;
2023-01-drips/src/Splits.sol::90 => uint128 collectable;
2023-01-drips/src/Splits.sol::126 => uint32 splitsWeight = 0;
2023-01-drips/src/Splits.sol::157 => uint32 splitsWeight = 0;
2023-01-drips/src/Splits.sol::228 => uint64 totalWeight = 0;
```

## [G-09] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, for example when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

```
2023-01-drips/src/Caller.sol::174 => uint256 currNonce = nonce[sender]++;
2023-01-drips/src/Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {
2023-01-drips/src/Drips.sol::247 => for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
2023-01-drips/src/Drips.sol::287 => for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
2023-01-drips/src/Drips.sol::358 => for (uint256 i = 0; i < squeezedNum; i++) {
2023-01-drips/src/Drips.sol::422 => for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
2023-01-drips/src/Drips.sol::428 => squeezedRevIdxs[squeezedNum++] = i;
2023-01-drips/src/Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips/src/Drips.sol::490 => for (; idx < receivers.length; idx++) {
2023-01-drips/src/Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Drips.sol::655 => state.currCycleConfigs++;
2023-01-drips/src/Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips/src/Drips.sol::745 => for (uint256 i = 0; i < configsLen; i++) {
2023-01-drips/src/Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Drips.sol::959 => currIdx++;
2023-01-drips/src/Drips.sol::962 => newIdx++;
2023-01-drips/src/DripsHub.sol::136 => driverId = dripsHubStorage.nextDriverId++;
2023-01-drips/src/DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {
2023-01-drips/src/ImmutableSplitsDriver.sol::59 => StorageSlot.getUint256Slot(_counterSlot).value++;
2023-01-drips/src/ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/NFTDriver.sol::93 => StorageSlot.getUint256Slot(_mintedTokensSlot).value++;
2023-01-drips/src/Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
```

## [G-10] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

use <x> = <x> + <y> or <x> = <x> - <y> instead to save gas

```
2023-01-drips/src/Drips.sol::253 => amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
2023-01-drips/src/Drips.sol::284 => toCycle -= receivableCycles;
2023-01-drips/src/Drips.sol::288 => amtPerCycle += state.amtDeltas[cycle].thisCycle;
2023-01-drips/src/Drips.sol::289 => receivedAmt += uint128(amtPerCycle);
2023-01-drips/src/Drips.sol::290 => amtPerCycle += state.amtDeltas[cycle].nextCycle;
2023-01-drips/src/Drips.sol::429 => amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);
2023-01-drips/src/Drips.sol::495 => amt += _drippedAmt(receiver.config.amtPerSec(), start, end);
2023-01-drips/src/Drips.sol::572 => balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
2023-01-drips/src/Drips.sol::755 => spent += _drippedAmt(amtPerSec, start, end);
2023-01-drips/src/Drips.sol::1053 => amtDelta.thisCycle += int128(fullCycle - nextCycle);
2023-01-drips/src/Drips.sol::1054 => amtDelta.nextCycle += int128(nextCycle);
2023-01-drips/src/DripsHub.sol::632 => totalBalances[erc20] += amt;
2023-01-drips/src/DripsHub.sol::636 => _dripsHubStorage().totalBalances[erc20] -= amt;
2023-01-drips/src/ImmutableSplitsDriver.sol::62 => weightSum += receivers[i].weight;
2023-01-drips/src/Splits.sol::99 => _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
2023-01-drips/src/Splits.sol::128 => splitsWeight += currReceivers[i].weight;
2023-01-drips/src/Splits.sol::159 => splitsWeight += currReceivers[i].weight;
2023-01-drips/src/Splits.sol::162 => splitAmt += currSplitAmt;
2023-01-drips/src/Splits.sol::167 => collectableAmt -= splitAmt;
2023-01-drips/src/Splits.sol::168 => balance.collectable += collectableAmt;
2023-01-drips/src/Splits.sol::235 => totalWeight += weight;
```

## [G-11] abi.encode() is less efficient than abi.encodePacked()

use abi.encodePacked() where possible to save gas

```
2023-01-drips/src/Caller.sol::176 => abi.encode(
2023-01-drips/src/Drips.sol::842 => return keccak256(abi.encode(receivers));
2023-01-drips/src/Drips.sol::858 => return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));
2023-01-drips/src/Splits.sol::278 => return keccak256(abi.encode(receivers));
```

## [G-12] Use custom errors rather than revert()/require() strings to save gas

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they're hitby avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

```
2023-01-drips/src/Caller.sol::108 => require(_authorized[sender].add(user), "Address already is authorized");
2023-01-drips/src/Caller.sol::116 => require(_authorized[sender].remove(user), "Address is not authorized");
2023-01-drips/src/Caller.sol::149 => require(isAuthorized(sender, _msgSender()), "Not authorized");
2023-01-drips/src/Caller.sol::173 => require(block.timestamp <= deadline, "Execution deadline expired");
2023-01-drips/src/Caller.sol::181 => require(signer == sender, "Invalid signature");
2023-01-drips/src/Drips.sol::220 => require(cycleSecs > 1, "Cycle length too low");
2023-01-drips/src/Drips.sol::454 => require(dripsHash == 0, "Drips history entry with hash and receivers");
2023-01-drips/src/Drips.sol::461 => require(historyHash == finalHistoryHash, "Invalid drips history");
2023-01-drips/src/Drips.sol::540 => require(timestamp >= state.updateTime, "Timestamp before last drips update");
2023-01-drips/src/Drips.sol::541 => require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list");
2023-01-drips/src/Drips.sol::622 => require(_hashDrips(currReceivers) == state.dripsHash, "Invalid current drips list");
2023-01-drips/src/Drips.sol::775 => require(receivers.length <= _MAX_DRIPS_RECEIVERS, "Too many drips receivers");
2023-01-drips/src/Drips.sol::780 => require(_isOrdered(receivers[i - 1], receiver), "Receivers not sorted");
2023-01-drips/src/Drips.sol::798 => require(amtPerSec != 0, "Drips receiver amtPerSec is zero");
2023-01-drips/src/DripsHub.sol::125 => require(driverAddress(driverId) == msg.sender, "Callable only by the driver");
2023-01-drips/src/DripsHub.sol::631 => require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");
2023-01-drips/src/ImmutableSplitsDriver.sol::64 => require(weightSum == totalSplitsWeight, "Invalid total receivers weight");
2023-01-drips/src/Managed.sol::47 => require(admin() == msg.sender, "Caller is not the admin");
2023-01-drips/src/Managed.sol::61 => require(!paused(), "Contract paused");
2023-01-drips/src/Managed.sol::67 => require(paused(), "Contract not paused");
2023-01-drips/src/Managed.sol::91 => require(_managedStorage().pausers.add(pauser), "Address already is a pauser");
2023-01-drips/src/Managed.sol::98 => require(_managedStorage().pausers.remove(pauser), "Address is not a pauser");
2023-01-drips/src/Splits.sol::227 => require(receivers.length <= _MAX_SPLITS_RECEIVERS, "Too many splits receivers");
2023-01-drips/src/Splits.sol::234 => require(weight != 0, "Splits receiver weight is zero");
2023-01-drips/src/Splits.sol::238 => require(prevUserId != userId, "Duplicate splits receivers");
2023-01-drips/src/Splits.sol::239 => require(prevUserId < userId, "Splits receivers not sorted by user ID");
2023-01-drips/src/Splits.sol::244 => require(totalWeight <= _TOTAL_SPLITS_WEIGHT, "Splits weights sum too high");
```

## [G-13] Prefix increments cheaper than Postfix increments

++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)
Saves 5 gas PER LOOP

```
2023-01-drips/src/Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {
2023-01-drips/src/Drips.sol::358 => for (uint256 i = 0; i < squeezedNum; i++) {
2023-01-drips/src/Drips.sol::422 => for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
2023-01-drips/src/Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips/src/Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips/src/Drips.sol::745 => for (uint256 i = 0; i < configsLen; i++) {
2023-01-drips/src/Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {
2023-01-drips/src/ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips/src/Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips/src/Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
```

## [G-14] Use bytes32 instead of string

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

```
2023-01-drips/src/Caller.sol::82 => string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("
```

## [G-15] Public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```
2023-01-drips/src/Caller.sol::114 => function unauthorize(address user) public {
2023-01-drips/src/Caller.sol::132 => function allAuthorized(address sender) public view returns (address[] memory authorized) {
2023-01-drips/src/Caller.sol::193 => function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {
2023-01-drips/src/DripsHub.sol::134 => function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
2023-01-drips/src/DripsHub.sol::152 => function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
2023-01-drips/src/DripsHub.sol::160 => function nextDriverId() public view returns (uint32 driverId) {
2023-01-drips/src/DripsHub.sol::169 => function cycleSecs() public view returns (uint32 cycleSecs_) {
2023-01-drips/src/DripsHub.sol::181 => function totalBalance(IERC20 erc20) public view returns (uint256 balance) {
2023-01-drips/src/Managed.sol::90 => function grantPauser(address pauser) public onlyAdmin {
2023-01-drips/src/Managed.sol::97 => function revokePauser(address pauser) public onlyAdmin {
2023-01-drips/src/Managed.sol::112 => function allPausers() public view returns (address[] memory pausersList) {
2023-01-drips/src/Managed.sol::128 => function unpause() public onlyAdminOrPauser whenPaused {
```

## [G-16] Not using the named return variables when a function returns, wastes deployment gas

It is not necessary to have both a named return and a return statement.

```
2023-01-drips/src/AddressDriver.sol::40 => function calcUserId(address userAddr) public view returns (uint256 userId) {
2023-01-drips/src/AddressDriver.sol::46 => function callerUserId() internal view returns (uint256 userId) {
2023-01-drips/src/Caller.sol::124 => function isAuthorized(address sender, address user) public view returns (bool authorized) {
2023-01-drips/src/Caller.sol::132 => function allAuthorized(address sender) public view returns (address[] memory authorized) {
2023-01-drips/src/Caller.sol::147 => returns (bytes memory returnData)
2023-01-drips/src/Caller.sol::171 => ) public payable returns (bytes memory returnData) {
2023-01-drips/src/Caller.sol::204 => returns (bytes memory returnData)
2023-01-drips/src/Drips.sol::303 => returns (uint32 cycles)
2023-01-drips/src/Drips.sol::475 => ) private view returns (uint128 squeezedAmt) {
2023-01-drips/src/Drips.sol::538 => ) internal view returns (uint128 balance) {
2023-01-drips/src/Drips.sol::685 => ) private view returns (uint32 maxEnd) {
2023-01-drips/src/Drips.sol::742 => ) private view returns (bool isEnough) {
2023-01-drips/src/Drips.sol::795 => returns (uint256 newConfigsLen)
2023-01-drips/src/Drips.sol::818 => returns (uint256 amtPerSec, uint256 start, uint256 end)
2023-01-drips/src/Drips.sol::837 => returns (bytes32 dripsHash)
2023-01-drips/src/Drips.sol::857 => ) internal pure returns (bytes32 dripsHistoryHash) {
2023-01-drips/src/Drips.sol::973 => returns (uint32 start, uint32 end)
2023-01-drips/src/Drips.sol::991 => ) private pure returns (uint32 start, uint32 end_) {
2023-01-drips/src/Drips.sol::1120 => function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
2023-01-drips/src/Drips.sol::1128 => function _currTimestamp() private view returns (uint32 timestamp) {
2023-01-drips/src/Drips.sol::1134 => function _currCycleStart() private view returns (uint32 timestamp) {
2023-01-drips/src/DripsHub.sol::145 => function driverAddress(uint32 driverId) public view returns (address driverAddr) {
2023-01-drips/src/DripsHub.sol::160 => function nextDriverId() public view returns (uint32 driverId) {
2023-01-drips/src/DripsHub.sol::169 => function cycleSecs() public view returns (uint32 cycleSecs_) {
2023-01-drips/src/DripsHub.sol::181 => function totalBalance(IERC20 erc20) public view returns (uint256 balance) {
2023-01-drips/src/DripsHub.sol::199 => returns (uint32 cycles)
2023-01-drips/src/DripsHub.sol::317 => function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
2023-01-drips/src/DripsHub.sol::331 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/DripsHub.sol::358 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/DripsHub.sol::372 => function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
2023-01-drips/src/DripsHub.sol::465 => ) public view returns (uint128 balance) {
2023-01-drips/src/DripsHub.sol::546 => function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {
2023-01-drips/src/DripsHub.sol::562 => ) public pure returns (bytes32 dripsHistoryHash) {
2023-01-drips/src/DripsHub.sol::587 => function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {
2023-01-drips/src/DripsHub.sol::598 => returns (bytes32 receiversHash)
2023-01-drips/src/DripsHub.sol::642 => function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {
2023-01-drips/src/ImmutableSplitsDriver.sol::36 => function nextUserId() public view returns (uint256 userId) {
2023-01-drips/src/Managed.sol::105 => function isPauser(address pauser) public view returns (bool isAddrPauser) {
2023-01-drips/src/Managed.sol::112 => function allPausers() public view returns (address[] memory pausersList) {
2023-01-drips/src/Managed.sol::117 => function paused() public view returns (bool isPaused) {
2023-01-drips/src/Managed.sol::136 => function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
2023-01-drips/src/NFTDriver.sol::50 => function nextTokenId() public view returns (uint256 tokenId) {
2023-01-drips/src/NFTDriver.sol::300 => function _msgData() internal view override(Context, ERC2771Context) returns (bytes calldata) {
2023-01-drips/src/Splits.sol::106 => function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
2023-01-drips/src/Splits.sol::120 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/Splits.sol::145 => returns (uint128 collectableAmt, uint128 splitAmt)
2023-01-drips/src/Splits.sol::176 => function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
2023-01-drips/src/Splits.sol::262 => function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {
2023-01-drips/src/Splits.sol::273 => returns (bytes32 receiversHash)
```

## [G-17] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

```
2023-01-drips/src/Caller.sol::88 => mapping(address => EnumerableSet.AddressSet) internal _authorized;
2023-01-drips/src/Caller.sol::90 => mapping(address => uint256) public nonce;
```

## [G-18] internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```
2023-01-drips/src/Drips.sol::73 => function dripId(DripsConfig config) internal pure returns (uint32) {
2023-01-drips/src/Drips.sol::83 => function start(DripsConfig config) internal pure returns (uint32) {
2023-01-drips/src/Drips.sol::88 => function duration(DripsConfig config) internal pure returns (uint32) {
2023-01-drips/src/Managed.sol::136 => function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
2023-01-drips/src/Splits.sol::106 => function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
2023-01-drips/src/Splits.sol::176 => function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
2023-01-drips/src/Splits.sol::185 => function _collect(uint256 userId, uint256 assetId) internal returns (uint128 amt) {
2023-01-drips/src/Splits.sol::199 => function _give(uint256 userId, uint256 receiver, uint256 assetId, uint128 amt) internal {
2023-01-drips/src/Splits.sol::212 => function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {
2023-01-drips/src/Splits.sol::262 => function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {
```

## [G-19] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```
2023-01-drips/src/Drips.sol::73 => function dripId(DripsConfig config) internal pure returns (uint32) {
2023-01-drips/src/Splits.sol::106 => function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
2023-01-drips/src/Splits.sol::176 => function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
2023-01-drips/src/Splits.sol::185 => function _collect(uint256 userId, uint256 assetId) internal returns (uint128 amt) {
2023-01-drips/src/Splits.sol::199 => function _give(uint256 userId, uint256 receiver, uint256 assetId, uint128 amt) internal {
2023-01-drips/src/Splits.sol::212 => function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {
```