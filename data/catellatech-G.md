# c4udit DRIPS Gas Report

## Files analyzed
- 2023-01-drips\src\AddressDriver.sol
- 2023-01-drips\src\Caller.sol
- 2023-01-drips\src\Drips.sol
- 2023-01-drips\src\DripsHub.sol
- 2023-01-drips\src\ImmutableSplitsDriver.sol
- 2023-01-drips\src\Managed.sol
- 2023-01-drips\src\NFTDriver.sol
- 2023-01-drips\src\Splits.sol

## Issues found


# USE CUSTOM ERRORS INSTEAD OF REVERT STRINGS TO SAVE GAS
Custom errors from Solidity 0.8.4 are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met)

Source: https://blog.soliditylang.org/2021/04/21/custom-errors/:

Starting from Solidity v0.8.4, there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., revert("Insufficient funds.");), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Custom errors are defined using the error statement, which can be used inside and outside of contracts (including interfaces and libraries).

🚨 I suggest replacing revert strings with custom errors.

#### Impact
Issue Information: [G001]

#### Findings:
```
2023-01-drips\src\Caller.sol::108 => require(_authorized[sender].add(user), "Address already is authorized");
2023-01-drips\src\Caller.sol::116 => require(_authorized[sender].remove(user), "Address is not authorized");
2023-01-drips\src\Caller.sol::149 => require(isAuthorized(sender, _msgSender()), "Not authorized");
2023-01-drips\src\Caller.sol::173 =>  require(block.timestamp <= deadline, "Execution deadline expired");
2023-01-drips\src\Caller.sol::181 =>  require(signer == sender, "Invalid signature");

2023-01-drips\src\Drips.sol:: 220 => require(cycleSecs > 1, "Cycle length too low");
2023-01-drips\src\Drips.sol:: 454 => require(dripsHash == 0, "Drips history entry with hash and receivers");
2023-01-drips\src\Drips.sol:: 461 => require(historyHash == finalHistoryHash, "Invalid drips history");
2023-01-drips\src\Drips.sol:: 540 =>  require(timestamp >= state.updateTime, "Timestamp before last drips update");
2023-01-drips\src\Drips.sol:: 541 => require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list");
2023-01-drips\src\Drips.sol:: 622 =>         require(_hashDrips(currReceivers) == state.dripsHash, "Invalid current drips list");
2023-01-drips\src\Drips.sol:: 775 => require(receivers.length <= _MAX_DRIPS_RECEIVERS, "Too many drips receivers");
2023-01-drips\src\Drips.sol:: 780 => require(_isOrdered(receivers[i - 1], receiver), "Receivers not sorted");
2023-01-drips\src\Drips.sol:: 798 =>  require(amtPerSec != 0, "Drips receiver amtPerSec is zero");

2023-01-drips\src\DripsHub.sol::125 => require(driverAddress(driverId) == msg.sender, "Callable only by the driver");
2023-01-drips\src\DripsHub.sol::631 => require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");

2023-01-drips\src\ImmutableSplitsDriver.sol::64 => require(weightSum == totalSplitsWeight, "Invalid total receivers weight");

2023-01-drips\src\Managed.sol::91 => require(_managedStorage().pausers.add(pauser), "Address already is a pauser");
2023-01-drips\src\Managed.sol::98 => require(_managedStorage().pausers.remove(pauser), "Address is not a pauser");  

2023-01-drips\src\Splits.sol::227 => require(receivers.length <= _MAX_SPLITS_RECEIVERS, "Too many splits receivers"); 
2023-01-drips\src\Splits.sol::234 => require(weight != 0, "Splits receiver weight is zero"); 
2023-01-drips\src\Splits.sol::238 => require(prevUserId != userId, "Duplicate splits receivers");
2023-01-drips\src\Splits.sol::239 => require(prevUserId < userId, "Splits receivers not sorted by user ID");
2023-01-drips\src\Splits.sol::244 => require(totalWeight <= _TOTAL_SPLITS_WEIGHT, "Splits weights sum too high");
2023-01-drips\src\Splits.sol::254 => require(
            _hashSplits(currReceivers) == _splitsHash(userId), "Invalid current splits receivers"
        );


```


#  NO NEED TO EXPLICITLY INITIALIZE VARIABLES WITH DEFAULT VALUES

If a variable is not set/initialized, it is assumed to have the default value (`0` for `uint`, `false` for `bool`, `address(0)` for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

As an example: `for (uint256 i = 0; i < numIterations; ++i) {` should be replaced with `for (uint256 i; i < numIterations; ++i) {`

#### Impact
Issue Information: [G002]

#### Findings:
```
2023-01-drips\src\Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {
2023-01-drips\src\Drips.sol::358 => for (uint256 i = 0; i < squeezedNum; i++) {
2023-01-drips\src\Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips\src\Drips.sol::478 => uint256 idx = 0;
2023-01-drips\src\Drips.sol::489 => uint256 amt = 0;
2023-01-drips\src\Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips\src\Drips.sol::744 => uint256 spent = 0;
2023-01-drips\src\Drips.sol::745 => for (uint256 i = 0; i < configsLen; i++) {
2023-01-drips\src\Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\Drips.sol::880 => uint256 currIdx = 0;
2023-01-drips\src\Drips.sol::881 => uint256 newIdx = 0;
2023-01-drips\src\DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {
2023-01-drips\src\ImmutableSplitsDriver.sol::60 => uint256 weightSum = 0;
2023-01-drips\src\ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\Splits.sol::126 => uint32 splitsWeight = 0;
2023-01-drips\src\Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips\src\Splits.sol::157 => uint32 splitsWeight = 0;
2023-01-drips\src\Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips\src\Splits.sol::228 => uint64 totalWeight = 0;
2023-01-drips\src\Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
```

# INCREMENTS CAN BE UNCHECKED

## Description 
In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

#### Impact
Issue Information: [G003]

#### Findings:

```
2023-01-drips\src\Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {

2023-01-drips\src\Drips.sol::358 => for (uint256 i = 0; i < squeezedNum; i++) {
2023-01-drips\src\Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips\src\Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips\src\Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {

2023-01-drips\src\DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {

2023-01-drips\src\ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {

2023-01-drips\src\Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips\src\Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips\src\Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
```

## The code would go from:

```
for (uint i = 0; i < numIterations; i++) {.....}
```

## to:

```
for (uint i = 0; i < numIterations;) {  
 // ...  
 unchecked { ++i; }  
}  
```

# PUBLIC FUNCTIONS TO EXTERNAL

## Description 
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### Impact
Issue Information: [G004]




# Cache Array Length Outside of Loop

#### Impact
Issue Information: [G005]
#### Findings:

```
2023-01-drips\src\Caller.sol::196 => for (uint256 i = 0; i < calls.length; i++) {

2023-01-drips\src\Drips.sol::422 => for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
2023-01-drips\src\Drips.sol::424 => if (drips.receivers.length != 0) {
2023-01-drips\src\Drips.sol::450 => for (uint256 i = 0; i < dripsHistory.length; i++) {
2023-01-drips\src\Drips.sol::453 => if (drips.receivers.length != 0) {
2023-01-drips\src\Drips.sol::479 => for (uint256 idxCap = receivers.length; idx < idxCap;) {
2023-01-drips\src\Drips.sol::490 => for (; idx < receivers.length; idx++) {
2023-01-drips\src\Drips.sol::563 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips\src\Drips.sol::777 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\Drips.sol::839 => if (receivers.length == 0) {

2023-01-drips\src\DripsHub.sol::613 => for (uint256 i = 0; i < userMetadata.length; i++) {

2023-01-drips\src\ImmutableSplitsDriver.sol::61 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\ImmutableSplitsDriver.sol::67 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);

2023-01-drips\src\NFTDriver.sol::69 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
2023-01-drips\src\NFTDriver.sol::86 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);

2023-01-drips\src\Splits.sol::127 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips\src\Splits.sol::158 => for (uint256 i = 0; i < currReceivers.length; i++) {
2023-01-drips\src\Splits.sol::231 => for (uint256 i = 0; i < receivers.length; i++) {
2023-01-drips\src\Splits.sol::275 => if (receivers.length == 0) {
```

## Recommendation
Here, I suggest storing the array’s length in a variable before the for-loop, and use it instead of .length


# Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G006]

#### Findings:
```
2023-01-drips\src\AddressDriver.sol::132 => if (balanceDelta > 0) {
2023-01-drips\src\Drips.sol::779 => if (i > 0) {
2023-01-drips\src\DripsHub.sol::245 => if (receivedAmt > 0) {
2023-01-drips\src\DripsHub.sol::279 => if (amt > 0) {
2023-01-drips\src\DripsHub.sol::520 => if (balanceDelta > 0) {
2023-01-drips\src\DripsHub.sol::532 => if (realBalanceDelta > 0) {
2023-01-drips\src\ImmutableSplitsDriver.sol::67 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);
2023-01-drips\src\NFTDriver.sol::69 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
2023-01-drips\src\NFTDriver.sol::86 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
2023-01-drips\src\NFTDriver.sol::197 => if (balanceDelta > 0) {
2023-01-drips\src\Splits.sol::237 => if (i > 0) {
```


# Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: [G007]
#### Findings:
```
2023-01-drips\src\Caller.sol::84 => bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));
2023-01-drips\src\Caller.sol::175 => bytes32 executeHash = keccak256(
2023-01-drips\src\Caller.sol::177 => callSignedTypeHash, sender, to, keccak256(data), msg.value, currNonce, deadline
2023-01-drips\src\Drips.sol::842 => return keccak256(abi.encode(receivers));
2023-01-drips\src\Drips.sol::858 => return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));
2023-01-drips\src\Managed.sol::137 => return bytes32(uint256(keccak256(bytes(name))) - 1);
2023-01-drips\src\Splits.sol::278 => return keccak256(abi.encode(receivers));
```


# Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G008]

#### Findings:
```
2023-01-drips\src\Drips.sol::480 => uint256 idxMid = (idx + idxCap) / 2;
2023-01-drips\src\Drips.sol::717 => uint256 end = (enoughEnd + notEnoughEnd) / 2;
```

# USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

#### Impact
Issue Information: [G009]

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one



#### Findings:
```
2023-01-drips\src\Splits.sol::117 => function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
2023-01-drips\src\Splits.sol::143 => function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)
2023-01-drips\src\Splits.sol::212 =>  function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal
2023-01-drips\src\Splits.sol::226 =>  function _assertSplitsValid(SplitsReceiver[] memory receivers, bytes32 receiversHash)
2023-01-drips\src\Splits.sol::250 =>  function _assertCurrSplits(uint256 userId, SplitsReceiver[] memory currReceivers)
2023-01-drips\src\Splits.sol::270 => function _hashSplits(SplitsReceiver[] memory receivers)
2023-01-drips\src\Splits.sol::250 =>  function _assertCurrSplits(uint256 userId, SplitsReceiver[] memory currReceivers)

2023-01-drips\src\Caller.sol::193 => function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {


2023-01-drips\src\Drips.sol::342 => function _squeezeDrips(
        uint256 userId,
        uint256 assetId,
        uint256 senderId,
        bytes32 historyHash,
        DripsHistory[] memory dripsHistory
    ) internal returns (uint128 amt) {
2023-01-drips\src\Drips.sol::392 => function _squeezeDripsResult(
        uint256 userId,
        uint256 assetId,
        uint256 senderId,
        bytes32 historyHash,
        DripsHistory[] memory dripsHistory
2023-01-drips\src\Drips.sol::444 => function _verifyDripsHistory(
        bytes32 historyHash,
        DripsHistory[] memory dripsHistory,
        bytes32 finalHistoryHash
    ) private pure returns (bytes32[] memory historyHashes) {
2023-01-drips\src\Drips.sol::533 =>function _balanceAt(
        uint256 userId,
        uint256 assetId,
        DripsReceiver[] memory receivers,
        uint32 timestamp
2023-01-drips\src\Drips.sol::555 => function _balanceAt(
        uint128 lastBalance,
        uint32 lastUpdate,
        uint32 maxEnd,
        DripsReceiver[] memory receivers,
        uint32 timestamp
2023-01-drips\src\Drips.sol::610 => function _setDrips(
        uint256 userId,
        uint256 assetId,
        DripsReceiver[] memory currReceivers,
        int128 balanceDelta,
        DripsReceiver[] memory newReceivers,
2023-01-drips\src\Drips.sol::610 =>     function _calcMaxEnd(
        uint128 balance,
        DripsReceiver[] memory receivers,
2023-01-drips\src\Drips.sol::737 => function _isBalanceEnough(
        uint256 balance,
        uint256[] memory configs,
2023-01-drips\src\Drips.sol::769 =>   function _buildConfigs(DripsReceiver[] memory receivers)
2023-01-drips\src\Drips.sol::792 =>  function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver)
2023-01-drips\src\Drips.sol::815 =>     function _getConfig(uint256[] memory configs, uint256 idx)
2023-01-drips\src\Drips.sol::834 => function _hashDrips(DripsReceiver[] memory receivers)
2023-01-drips\src\Drips.sol::872 =>  function _updateReceiverStates(
        mapping(uint256 => DripsState) storage states,
        DripsReceiver[] memory currReceivers,
2023-01-drips\src\Drips.sol::834 => function _hashDrips(DripsReceiver[] memory receivers)

```

# <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

#### Impact
Issue Information: [G0010]

#### Findings:
```
2023-01-drips\src\Drips.sol::253 =>   amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
2023-01-drips\src\Drips.sol::288 =>   amtPerCycle += state.amtDeltas[cycle].thisCycle;
2023-01-drips\src\Drips.sol::289 =>   receivedAmt += uint128(amtPerCycle);
2023-01-drips\src\Drips.sol::290 =>   amtPerCycle += state.amtDeltas[cycle].nextCycle;
2023-01-drips\src\Drips.sol::755 =>   spent += _drippedAmt(amtPerSec, start, end);
2023-01-drips\src\Drips.sol::1053 =>  amtDelta.thisCycle += int128(fullCycle - nextCycle);
2023-01-drips\src\Drips.sol::1054 =>  amtDelta.nextCycle += int128(nextCycle);


2023-01-drips\src\DripsHub.sol::632 =>  totalBalances[erc20] += amt;

2023-01-drips\src\ImmutableSplitsDriver.sol::62 =>  weightSum += receivers[i].weight;

2023-01-drips\src\Splits.sol::128 => splitsWeight += currReceivers[i].weight;
2023-01-drips\src\Splits.sol::159 =>  splitsWeight += currReceivers[i].weight;
2023-01-drips\src\Splits.sol::162 =>  splitAmt += currSplitAmt;
2023-01-drips\src\Splits.sol::168 =>  balance.collectable += collectableAmt;
2023-01-drips\src\Splits.sol::235 =>  totalWeight += weight;

```