### 1st BUG
Cache Array Length Outside of Loop

#### Impact
Issue Information: 
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Example
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}

#### Findings:
```
2023-01-drips-main/src/Caller.sol::194 => returnData = new bytes[](calls.length);
2023-01-drips-main/src/Drips.sol::220 => require(cycleSecs > 1, "Cycle length too low");
2023-01-drips-main/src/Drips.sol::362 => squeezedHistoryHashes[i] = historyHashes[historyHashes.length - revIdx];
2023-01-drips-main/src/Drips.sol::418 => squeezedRevIdxs = new uint256[](dripsHistory.length);
2023-01-drips-main/src/Drips.sol::423 => DripsHistory memory drips = dripsHistory[dripsHistory.length - i];
2023-01-drips-main/src/Drips.sol::424 => if (drips.receivers.length != 0) {
2023-01-drips-main/src/Drips.sol::449 => historyHashes = new bytes32[](dripsHistory.length);
2023-01-drips-main/src/Drips.sol::453 => if (drips.receivers.length != 0) {
2023-01-drips-main/src/Drips.sol::664 => for (uint256 i = 0; i < newReceivers.length; i++) {
2023-01-drips-main/src/Drips.sol::775 => require(receivers.length <= _MAX_DRIPS_RECEIVERS, "Too many drips receivers");
2023-01-drips-main/src/Drips.sol::776 => configs = new uint256[](receivers.length);
2023-01-drips-main/src/Drips.sol::839 => if (receivers.length == 0) {
2023-01-drips-main/src/Drips.sol::883 => bool pickCurr = currIdx < currReceivers.length;
2023-01-drips-main/src/Drips.sol::890 => bool pickNew = newIdx < newReceivers.length;
2023-01-drips-main/src/ImmutableSplitsDriver.sol::67 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);
2023-01-drips-main/src/NFTDriver.sol::69 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
2023-01-drips-main/src/NFTDriver.sol::86 => if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
2023-01-drips-main/src/Splits.sol::227 => require(receivers.length <= _MAX_SPLITS_RECEIVERS, "Too many splits receivers");
2023-01-drips-main/src/Splits.sol::275 => if (receivers.length == 0) {
```
#### Tools used
Manual VS code audit

### 2nd BUG
Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: 
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

Example
🤦 Bad:

bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
🚀 Good:

bytes32 public immutable GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");

#### Findings:
```
2023-01-drips-main/src/Caller.sol::84 => bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));
2023-01-drips-main/src/Caller.sol::175 => bytes32 executeHash = keccak256(
2023-01-drips-main/src/Caller.sol::177 => callSignedTypeHash, sender, to, keccak256(data), msg.value, currNonce, deadline
2023-01-drips-main/src/Drips.sol::842 => return keccak256(abi.encode(receivers));
2023-01-drips-main/src/Drips.sol::858 => return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));
2023-01-drips-main/src/Managed.sol::137 => return bytes32(uint256(keccak256(bytes(name))) - 1);
2023-01-drips-main/src/Splits.sol::278 => return keccak256(abi.encode(receivers));
```
#### Tools used
Manual VS code audit