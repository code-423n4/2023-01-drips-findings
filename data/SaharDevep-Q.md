# Audit Report

## Files analyzed
src/AddressDriver.sol
src/Caller.sol
src/Drips.sol
src/DripsHub.sol
src/ImmutableSplitsDriver.sol
src/Managed.sol
src/NFTDriver.sol
src/Splits.sol

## Summery
[L01] FLOATING PRAGMA VERSION
[N01] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

## Issues Found

### [L01] FLOATING PRAGMA VERSION
While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different evm compilation that is ultimately deployed on the blockchain.
Please consider locking the pragma in all files.

#### Tools used
Manual

### [N01] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

#### Findings:
```
src/AddressDriver.sol::40 => function calcUserId(address userAddr) public view returns (uint256 userId) {
src/AddressDriver.sol::46 => function callerUserId() internal view returns (uint256 userId) {
src/Caller.sol::124 => function isAuthorized(address sender, address user) public view returns (bool authorized) {
src/Caller.sol::132 => function allAuthorized(address sender) public view returns (address[] memory authorized) {
src/Caller.sol::144 => unction callAs(address sender, address to, bytes memory data)
src/Caller.sol::164 => function callSigned(
src/Caller.sol::202 => function _call(address sender, address to, bytes memory data, uint256 value)
src/Drips.sol::300 => function _receivableDripsCycles(uint256 userId, uint256 assetId)
src/Drips.sol::470 => function _squeezedAmt(
src/Drips.sol::680 => function _calcMaxEnd(
src/Drips.sol::737 => function _isBalanceEnough(
src/Drips.sol::792 => function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver)
src/Drips.sol::815 => function _getConfig(uint256[] memory configs, uint256 idx)
src/Drips.sol::834 => function _hashDrips(DripsReceiver[] memory receivers)
src/Drips.sol::852 => function _hashDripsHistory(
src/Drips.sol::970 => function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)
src/Drips.sol::1120 => function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
src/Drips.sol::1128 => function _currTimestamp() private view returns (uint32 timestamp) {
src/Drips.sol::1134 => function _currCycleStart() private view returns (uint32 timestamp) {
src/DripsHub.sol::145 => function driverAddress(uint32 driverId) public view returns (address driverAddr) {
src/DripsHub.sol::160 => function nextDriverId() public view returns (uint32 driverId) {
src/DripsHub.sol::169 => function cycleSecs() public view returns (uint32 cycleSecs_) {
src/DripsHub.sol::181 => function totalBalance(IERC20 erc20) public view returns (uint256 balance) {
src/DripsHub.sol::196 => function receivableDripsCycles(uint256 userId, IERC20 erc20)
src/DripsHub.sol::317 => function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
src/DripsHub.sol::328 => function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
src/DripsHub.sol::355 => function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)
src/DripsHub.sol::372 => function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
src/DripsHub.sol::432 => function dripsState(uint256 userId, IERC20 erc20)
src/DripsHub.sol::460 => function balanceAt(
src/DripsHub.sol::546 => function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {
src/DripsHub.sol::557 => function hashDripsHistory(
src/DripsHub.sol::587 => function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {
src/DripsHub.sol::595 => function hashSplits(SplitsReceiver[] memory receivers)
src/DripsHub.sol::642 => function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {
src/ImmutableSplitsDriver.sol::36 => function nextUserId() public view returns (uint256 userId) {
src/Managed.sol::105 => function isPauser(address pauser) public view returns (bool isAddrPauser) {
src/Managed.sol::112 => function allPausers() public view returns (address[] memory pausersList) {
src/Managed.sol::136 => function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
src/NFTDriver.sol::50 => function nextTokenId() public view returns (uint256 tokenId) {
src/Splits.sol::106 => function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
src/Splits.sol::176 => function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
src/Splits.sol::262 => function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {
src/Splits.sol::270 => function _hashSplits(SplitsReceiver[] memory receivers)
```

#### Tools used
Manual