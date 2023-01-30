# Report

## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1] | State variables are not packed together | 2 |
| [GAS-2] | Functions that will be called more often (like public/external) should be higher in a file structure | 7 |
| [GAS-3] | Lack of unchecked keyword inside for loops | 9 |
### [GAS-1] State variables are not packed together
State variables should be stored together to save gas, e.g. uint8 with uint32, etc. Moreover if uint8 state variable will be called and later on we try to access uint32 state variable (which is in the same storage slot) it will be treated as a warm storage (even if its our first contact with that variable) and thus it will only cost 100 gas instead of 2100 gas to use it.

*Instances (2)*:
```solidity
File: DripsHub.sol

56:    uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;
58:    uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
60:    uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
62:    uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
64:    uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;

```
```solidity
File: Drips.sol

106:    uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;
108:    uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
110:    uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
112:    uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
117:    uint32 internal immutable _cycleSecs;

```
### [GAS-2] Functions that will be called more often (like public/external) should be higher in the file structure
Functions that will be called more often (like public/external) should be at the top of the contract file to save gas because for every function inside the contract the next one to call will cost 22 gas more (method id grow with every function). It's crucial especially in a very long contracts.

*Instances (7)*:
```solidity
File: DripsHub.sol

510:   function setDrips(uint256 userId, IERC20 erc20, DripsReceiver[] memory currReceivers, int128 balanceDelta, DripsReceiver[] memory newReceivers, // slither-disable-next-line similar-names uint32 maxEndHint1, uint32 maxEndHint2) public whenNotPaused onlyDriver(userId) returns (int128 realBalanceDelta)
546:   function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash)
557:   function hashDripsHistory(bytes32 oldDripsHistoryHash, bytes32 dripsHash, uint32 updateTime, uint32 maxEnd) public pure returns (bytes32 dripsHistoryHash)
576:   function setSplits(uint256 userId, SplitsReceiver[] memory receivers) public whenNotPaused onlyDriver(userId)
587:   function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash)
595:   function hashSplits(SplitsReceiver[] memory receivers) public pure returns (bytes32 receiversHash)
608:   function emitUserMetadata(uint256 userId, UserMetadata[] calldata userMetadata) public whenNotPaused onlyDriver(userId)

```
### [GAS-3] Lack of unchecked keyword inside for loops
Unchecked keyword should be used inside the for loop for incrementor part if there is no risk of overflowing a value. It saves about 30-40 gas per loop.

*Instances (9)*:
```solidity
File: Drips.sol

247:   for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
287:   for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
358:   for (uint256 i = 0; i < squeezedNum; i++) {
664:   for (uint256 i = 0; i < newReceivers.length; i++) {

```
```solidity
File: Splits.sol

127:   for (uint256 i = 0; i < currReceivers.length; i++) {
158:   for (uint256 i = 0; i < currReceivers.length; i++) {
231:   for (uint256 i = 0; i < receivers.length; i++) {

```
```solidity
File: DripsHub.sol

613:   for (uint256 i = 0; i < userMetadata.length; i++) {

```
```solidity
File: ImmutableSplitsDriver.sol

61:    for (uint256 i = 0; i < receivers.length; i++) {

```