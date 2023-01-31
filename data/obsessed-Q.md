# Report

## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1] | Named parameters for the return statement are not used inside function | 15 |
| [NC-2] | Insufficient Coverage | 1 |
| [NC-3] | Use safeMint instead of mint | 1 |

### [NC-1] Named parameters for the return statement are not used inside function
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

### [NC-2] Insufficient Coverage
The test coverage rate of the project is not enough. Testing all functions is best practice in terms of security criteria.

```bash
| File                          | % Lines          | % Statements     | % Branches       | % Funcs          |
|-------------------------------|------------------|------------------|------------------|------------------|
| src/AddressDriver.sol         | 100.00% (16/16)  | 100.00% (16/16)  | 100.00% (6/6)    | 87.50% (7/8)     |
| src/Caller.sol                | 100.00% (22/22)  | 100.00% (30/30)  | 100.00% (10/10)  | 100.00% (8/8)    |
| src/Drips.sol                 | 89.05% (244/274) | 89.27% (283/317) | 66.98% (71/106)  | 83.33% (30/36)   |
| src/DripsHub.sol              | 100.00% (58/58)  | 100.00% (63/63)  | 100.00% (14/14)  | 100.00% (31/31)  |
| src/ImmutableSplitsDriver.sol | 100.00% (10/10)  | 100.00% (13/13)  | 75.00% (3/4)     | 100.00% (2/2)    |
| src/Managed.sol               | 94.12% (16/17)   | 94.12% (16/17)   | 100.00% (4/4)    | 100.00% (12/12)  |
| src/NFTDriver.sol             | 87.10% (27/31)   | 87.88% (29/33)   | 80.00% (8/10)    | 94.44% (17/18)   |
| src/Splits.sol                | 100.00% (64/64)  | 100.00% (71/71)  | 68.18% (15/22)   | 100.00% (13/13)  |

```

### [NC-3] Use safeMint instead of mint
According to openzepplin’s ERC721, the use of _mint is discouraged, use safeMint whenever possible.

*Instances (1)*:

```solidity
File: NFTDriver.sol

62:   function mint(address to, UserMetadata[] calldata userMetadata) public whenNotPaused returns (uint256 tokenId) {

```