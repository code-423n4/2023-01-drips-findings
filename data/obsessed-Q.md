# Report

## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1] | Named parameter/s in the return statement of a function are not used | 15 |

### [NC-1] Named parameter/s in the return statement of a function are not used
Named parameter/s in the return statement of a function are not used and thus it creates more confusion and don't improve overall readability.

*Instances (15)*:
```solidity
File: DripsHub.sol

181:   function totalBalance(IERC20 erc20) public view returns (uint256 balance) {
196:   function receivableDripsCycles(uint256 userId, IERC20 erc20) public view returns (uint32 cycles) {
328:   function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount) public view returns (uint128 collectableAmt, uint128 splitAmt) {
355:   function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers) public whenNotPaused returns (uint128 collectableAmt, uint128 splitAmt) {
372:   function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
432:   function dripsState(uint256 userId, IERC20 erc20) public view returns (bytes32 dripsHash, bytes32 dripsHistoryHash, uint32 updateTime, uint128 balance, uint32 maxEnd) {
460:   function balanceAt(uint256 userId, IERC20 erc20, DripsReceiver[] memory receivers, uint32 timestamp) public view returns (uint128 balance) {
546:   function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {
557:   function hashDripsHistory(bytes32 oldDripsHistoryHash, bytes32 dripsHash, uint32 updateTime, uint32 maxEnd) public pure returns (bytes32 dripsHistoryHash) {
587:   function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {
595:   function hashSplits(SplitsReceiver[] memory receivers) public pure returns (bytes32 receiversHash) {

```
```solidity
File: NFTDriver.sol

50:   function nextTokenId() public view returns (uint256 tokenId) {

```
```solidity
File: AddressDriver.sol

40:   function calcUserId(address userAddr) public view returns (uint256 userId) {
46:   function callerUserId() internal view returns (uint256 userId) {

```
```solidity
File: ImmutableSplitsDriver.sol

36:   function nextUserId() public view returns (uint256 userId) {

```