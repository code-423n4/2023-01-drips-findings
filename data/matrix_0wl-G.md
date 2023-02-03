# Summary

## Gas Optimizations

|        | Issue                                                                                                 |   Instances   |
| ------ | :---------------------------------------------------------------------------------------------------- | :-----------: |
| GAS-1  | `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables |       7       |
| GAS-2  | Unsafe cast                                                                                           |       7       |
| GAS-3  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                           |       4       |
| GAS-4  | Setting the constructor to payable                                                                    |       8       |
| GAS-5  | DoS With Block Gas Limit                                                                              |       9       |
| GAS-6  | Internal functions only called once can be inlined to save gas                                        |       4       |
| GAS-7  | Functions guaranteed to revert when called by normal users can be marked payable                      |      16       |
| GAS-8  | Optimize names to save gas                                                                            |       7       |
| GAS-9  | The increment in for loop postcondition can be made unchecked                                         |      16       |
| GAS-10 | Proper data types                                                                                     | all contracts |
| GAS-11 | `require()` or `revert()` statements that check input arguments should be at the top of the function  |       1       |
| GAS-12 | Functions not used internally could be marked external                                                |      26       |
| GAS-13 | Using storage instead of memory for structs/arrays saves gas                                          |       5       |
| GAS-14 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs                                         | all contracts |
| GAS-15 | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS                   |       3       |
| GAS-16 | NOT USING THE NAMED RETURN VARIABLES IN THE FUNCTION IS CONFUSING                                     |      10       |

### [GAS-1] `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables

Using compound assignment operators for state variables (like `State += X` or `State -= X` …) it’s more expensive than using operator assignment (like `State = State + X` or `State = State - X` …).

##### **Proof Of Concept**

```solidity
File: Drips.sol

253:                 amtDeltas[toCycle].thisCycle += finalAmtPerCycle;

572:             balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

1053:             amtDelta.thisCycle += int128(fullCycle - nextCycle);

1054:             amtDelta.nextCycle += int128(nextCycle);

```

```solidity
File: DripsHub.sol

632:         totalBalances[erc20] += amt;

636:         _dripsHubStorage().totalBalances[erc20] -= amt;

```

```solidity
File: Splits.sol

99:         _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;

```

### [GAS-2] Unsafe cast

Use openzeppilin’s safeCast in:

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

133:             _transferFromCaller(erc20, uint128(balanceDelta));

```

```solidity
File: Drips.sol

112:     uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);

289:             receivedAmt += uint128(amtPerCycle);

572:             balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

```

```solidity
File: DripsHub.sol

521:             _increaseTotalBalance(erc20, uint128(balanceDelta));

533:             erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));

```

```solidity
File: NFTDriver.sol

198:             _transferFromCaller(erc20, uint128(balanceDelta));

```

### [GAS-3] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

77:         _transferFromCaller(erc20, amt);

133:             _transferFromCaller(erc20, uint128(balanceDelta));

```

```solidity
File: NFTDriver.sol

198:             _transferFromCaller(erc20, uint128(balanceDelta));

285:     function _transferFromCaller(IERC20 erc20, uint128 amt) internal {

```

### [GAS-4] Setting the constructor to payable

Saves ~13 gas per instance

##### **Proof Of Concept**

```solidity
File: AddressDriver.sol

30:     constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)

```

```solidity
File: Drips.sol

219:     constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {

```

```solidity
File: DripsHub.sol

109:     constructor(uint32 cycleSecs_)

```

```solidity
File: ImmutableSplitsDriver.sol

28:     constructor(DripsHub _dripsHub, uint32 _driverId) {

```

```solidity
File: Managed.sol

73:     constructor() {

158:     constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0)) {

```

```solidity
File: NFTDriver.sol

32:     constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)

```

```solidity
File: Splits.sol

94:     constructor(bytes32 splitsStorageSlot) {

```

### [GAS-5] DoS With Block Gas Limit

When smart contracts are deployed or functions inside them are called, the execution of these actions always requires a certain amount of gas, based of how much computation is needed to complete them. The Ethereum network specifies a block gas limit and the sum of all transactions included in a block can not exceed the threshold.

Programming patterns that are harmless in centralized applications can lead to Denial of Service conditions in smart contracts when the cost of executing a function exceeds the block gas limit. Modifying an array of unknown size, that increases in size over time, can lead to such a Denial of Service condition.

https://swcregistry.io/docs/SWC-128

#### **Proof Of Concept**

```solidity
File: Caller.sol

193:     function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {

```

```solidity
File: Drips.sol

397:         DripsHistory[] memory dripsHistory

446:         DripsHistory[] memory dripsHistory,

559:         DripsReceiver[] memory receivers,

769:     function _buildConfigs(DripsReceiver[] memory receivers)

```

```solidity
File: ImmutableSplitsDriver.sol

53:     function createSplits(SplitsReceiver[] calldata receivers, UserMetadata[] calldata userMetadata)

```

```solidity
File: Splits.sol

117:     function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)

143:     function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)

226:     function _assertSplitsValid(SplitsReceiver[] memory receivers, bytes32 receiversHash) private {


```

#### Recommended Mitigation Steps:

Caution is advised when you expect to have large arrays that grow over time. Actions that require looping across the entire data structure should be avoided.

If you absolutely must loop over an array of unknown size, then you should plan for it to potentially take multiple blocks, and therefore require multiple transactions.

### [GAS-6] Internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

#### **Proof Of Concept**

```solidity
File: Drips.sol

83:     function start(DripsConfig config) internal pure returns (uint32) {

88:     function duration(DripsConfig config) internal pure returns (uint32) {

94:     function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {

```

```solidity
File: Managed.sol

136:     function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {

```

### [GAS-7] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: DripsHub.sol

386:             function collect(uint256 userId, IERC20 erc20)

409:             function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)

510:         function setDrips(

576:             function setSplits(uint256 userId, SplitsReceiver[] memory receivers)

608:             function emitUserMetadata(uint256 userId, UserMetadata[] calldata userMetadata)

```

```solidity
File: Managed.sol

84:     function changeAdmin(address newAdmin) public onlyAdmin {

90:     function grantPauser(address pauser) public onlyAdmin {

97:     function revokePauser(address pauser) public onlyAdmin {

122:     function pause() public onlyAdminOrPauser whenNotPaused {

128:     function unpause() public onlyAdminOrPauser whenPaused {

151:     function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {

```

```solidity
File: NFTDriver.sol

109:         function collect(uint256 tokenId, IERC20 erc20, address transferTo)

133:         function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)

186:         function setDrips(

220:         function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)

235:         function emitUserMetadata(uint256 tokenId, UserMetadata[] calldata userMetadata)

```

#### Recommended Mitigation Steps:

Functions guaranteed to revert when called by normal users can be marked payable.

### [GAS-8] Optimize names to save gas

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

19: contract AddressDriver is Managed, ERC2771Context {

```

```solidity
File: Caller.sol

81: contract Caller is EIP712("Caller", "1"), ERC2771Context(address(this)) {

```

```solidity
File: DripsHub.sol

53: contract DripsHub is Managed, Drips, Splits {

```

```solidity
File: ImmutableSplitsDriver.sol

11: contract ImmutableSplitsDriver is Managed {

```

```solidity
File: Managed.sol

18: abstract contract Managed is UUPSUpgradeable {

157: contract ManagedProxy is ERC1967Proxy {

```

```solidity
File: NFTDriver.sol

19: contract NFTDriver is ERC721Burnable, ERC2771Context, Managed {

```

### [GAS-9] The increment in for loop postcondition can be made unchecked

This is only relevant if we are using the default solidity checked arithmetic such in this case.

The for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of `i` to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

#### **Proof Of Concept**

```solidity
File: Caller.sol

196:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: Drips.sol

247:             for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

287:         for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

358:         for (uint256 i = 0; i < squeezedNum; i++) {

422:         for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

450:         for (uint256 i = 0; i < dripsHistory.length; i++) {

490:         for (; idx < receivers.length; idx++) {

563:         for (uint256 i = 0; i < receivers.length; i++) {

664:             for (uint256 i = 0; i < newReceivers.length; i++) {

745:             for (uint256 i = 0; i < configsLen; i++) {

777:             for (uint256 i = 0; i < receivers.length; i++) {

```

```solidity
File: DripsHub.sol

613:         for (uint256 i = 0; i < userMetadata.length; i++) {

```

```solidity
File: ImmutableSplitsDriver.sol

61:         for (uint256 i = 0; i < receivers.length; i++) {

```

```solidity
File: Splits.sol

127:         for (uint256 i = 0; i < currReceivers.length; i++) {

158:         for (uint256 i = 0; i < currReceivers.length; i++) {

231:         for (uint256 i = 0; i < receivers.length; i++) {

```

### [GAS-10] Proper data types

In Solidity, some data types are more expensive than others. It’s important to be aware of the most efficient type that can be used. Here are a few rules about data types.

Type `uint` should be used in place of type string whenever possible.

Type `uint256` takes less gas to store than `uint8`

Type `bytes` should be used over `byte[]`

If the length of bytes can be limited, use the lowest amount possible from `bytes1` to `bytes32`.

Type `bytes32` is cheaper to use than type `string` and `bytes`.

If data can fit into 32 bytes, then you should use `bytes32` datatype rather than bytes or strings as it is cheaper in solidity.

[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

Fixed size variables are always cheaper than dynamic ones.

[Source](https://medium.com/coinmonks/gas-optimization-in-solidity-part-i-variables-9d5775e43dde)

Most of the time it will be better to use a mapping instead of an array because of its cheaper operations.

However, an array can be the correct choice when using smaller data types. Array elements are packed like other storage variables and the reduced storage space can outweigh the cost of an array’s more expensive operations. This is most useful when working with large arrays.

[Source](https://medium.com/coinmonks/gas-optimization-in-solidity-part-i-variables-9d5775e43dde)

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

25:     uint32 public immutable driverId;

30:     constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)

41:         return (uint256(driverId) << 224) | uint160(userAddr);

60:     function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {

76:     function give(uint256 receiver, IERC20 erc20, uint128 amt) public whenNotPaused {

124:         DripsReceiver[] calldata currReceivers,

126:         DripsReceiver[] calldata newReceivers,

128:         uint32 maxEndHint1,

129:         uint32 maxEndHint2,

133:             _transferFromCaller(erc20, uint128(balanceDelta));

145:             erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));

158:     function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {

166:     function emitUserMetadata(UserMetadata[] calldata userMetadata) public whenNotPaused {

170:     function _transferFromCaller(IERC20 erc20, uint128 amt) internal {

```

```solidity
File: Caller.sol

17:     bytes data;

84:     bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));

132:     function allAuthorized(address sender) public view returns (address[] memory authorized) {

144:     function callAs(address sender, address to, bytes memory data)

147:         returns (bytes memory returnData)

167:         bytes memory data,

168:         uint256 deadline,

169:         bytes32 r,

170:         bytes32 sv

171:     ) public payable returns (bytes memory returnData) {

193:     function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {

202:     function _call(address sender, address to, bytes memory data, uint256 value)

204:         returns (bytes memory returnData)

```

```solidity
File: Drips.sol

26:     DripsReceiver[] receivers;

28:     uint32 updateTime;

30:     uint32 maxEnd;

60:     function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)

74:         return uint32(DripsConfig.unwrap(config) >> 224);

79:         return uint160(DripsConfig.unwrap(config) >> 64);

84:         return uint32(DripsConfig.unwrap(config) >> 32);

89:         return uint32(DripsConfig.unwrap(config));

108:     uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;

117:     uint32 internal immutable _cycleSecs;

135:         uint128 balance,

136:         uint32 maxEnd

144:         bytes32 indexed receiversHash, uint256 indexed userId, DripsConfig config

153:         uint256 indexed userId, uint256 indexed assetId, uint128 amt, uint32 receivableCycles

169:         uint128 amt,

189:         uint32 nextReceivableCycle;

191:         uint32 updateTime;

193:         uint32 maxEnd;

195:         uint128 balance;

197:         uint32 currCycleConfigs;

219:     constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {

233:     function _receiveDrips(uint256 userId, uint256 assetId, uint32 maxCycles)

235:         returns (uint128 receivedAmt)

237:         uint32 receivableCycles;

238:         uint32 fromCycle;

239:         uint32 toCycle;

247:             for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

270:     function _receiveDripsResult(uint256 userId, uint256 assetId, uint32 maxCycles)

274:             uint128 receivedAmt,

275:             uint32 receivableCycles,

276:             uint32 fromCycle,

277:             uint32 toCycle,

287:         for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

289:             receivedAmt += uint128(amtPerCycle);

303:         returns (uint32 cycles)

305:         (uint32 fromCycle, uint32 toCycle) = _receivableDripsCyclesRange(userId, assetId);

317:         returns (uint32 fromCycle, uint32 toCycle)

347:         DripsHistory[] memory dripsHistory

350:         uint256[] memory squeezedRevIdxs;

351:         bytes32[] memory historyHashes;

355:         bytes32[] memory squeezedHistoryHashes = new bytes32[](squeezedNum);

357:         uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];

365:         uint32 cycleStart = _currCycleStart();

397:         DripsHistory[] memory dripsHistory

402:             uint128 amt,

404:             uint256[] memory squeezedRevIdxs,

405:             bytes32[] memory historyHashes,

418:         squeezedRevIdxs = new uint256[](dripsHistory.length);

419:         uint32[2 ** 32] storage nextSqueezed =

421:         uint32 squeezeEndCap = _currTimestamp();

425:                 uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];

446:         DripsHistory[] memory dripsHistory,

449:         historyHashes = new bytes32[](dripsHistory.length);

473:         uint32 squeezeStartCap,

474:         uint32 squeezeEndCap

476:         DripsReceiver[] memory receivers = dripsHistory.receivers;

487:         uint32 updateTime = dripsHistory.updateTime;

488:         uint32 maxEnd = dripsHistory.maxEnd;

493:             (uint32 start, uint32 end) =

497:         return uint128(amt);

514:             uint32 updateTime,

515:             uint128 balance,

516:             uint32 maxEnd

536:         DripsReceiver[] memory receivers,

537:         uint32 timestamp

556:         uint128 lastBalance,

557:         uint32 lastUpdate,

558:         uint32 maxEnd,

559:         DripsReceiver[] memory receivers,

560:         uint32 timestamp

565:             (uint32 start, uint32 end) = _dripsRange({

572:             balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

611:         uint256 userId,

612:         uint256 assetId,

613:         DripsReceiver[] memory currReceivers,

615:         DripsReceiver[] memory newReceivers,

617:         uint32 maxEndHint1,

618:         uint32 maxEndHint2

623:         uint32 lastUpdate = state.updateTime;

624:         uint128 newBalance;

625:         uint32 newMaxEnd;

627:             uint32 currMaxEnd = state.maxEnd;

636:             newBalance = uint128(currBalance + realBalanceDelta);

681:         uint128 balance,

682:         DripsReceiver[] memory receivers,

683:         uint32 hint1,

684:         uint32 hint2

687:             (uint256[] memory configs, uint256 configsLen) = _buildConfigs(receivers);

692:                 return uint32(enoughEnd);

697:                 return uint32(notEnoughEnd);

719:                     return uint32(end);

739:         uint256[] memory configs,

769:     function _buildConfigs(DripsReceiver[] memory receivers)

772:         returns (uint256[] memory configs, uint256 configsLen)

776:             configs = new uint256[](receivers.length);

792:     function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver)

800:             _dripsRangeInFuture(receiver, _currTimestamp(), type(uint32).max);

815:     function _getConfig(uint256[] memory configs, uint256 idx)

825:         return (val >> 64, uint32(val >> 32), uint32(val));

834:     function _hashDrips(DripsReceiver[] memory receivers)

855:         uint32 updateTime,

856:         uint32 maxEnd

874:         DripsReceiver[] memory currReceivers,

875:         uint32 lastUpdate,

876:         uint32 currMaxEnd,

877:         DripsReceiver[] memory newReceivers,

878:         uint32 newMaxEnd

912:                 (uint32 currStart, uint32 currEnd) =

914:                 (uint32 newStart, uint32 newEnd) =

917:                     int256 amtPerSec = int256(uint256(currRecv.config.amtPerSec()));

923:                 uint32 currStartCycle = _cycleOf(currStart);

924:                 uint32 newStartCycle = _cycleOf(newStart);

936:                 (uint32 start, uint32 end) = _dripsRangeInFuture(currRecv, lastUpdate, currMaxEnd);

938:                 int256 amtPerSec = int256(uint256(currRecv.config.amtPerSec()));

944:                 (uint32 start, uint32 end) =

946:                 int256 amtPerSec = int256(uint256(newRecv.config.amtPerSec()));

949:                 uint32 startCycle = _cycleOf(start);

970:     function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)

973:         returns (uint32 start, uint32 end)

975:         return _dripsRange(receiver, updateTime, maxEnd, _currTimestamp(), type(uint32).max);

987:         uint32 updateTime,

988:         uint32 maxEnd,

989:         uint32 startCap,

990:         uint32 endCap

997:         uint40 end = uint40(start) + receiver.config.duration();

1012:         return (start, uint32(end));

1020:     function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)

1128:     function _currTimestamp() private view returns (uint32 timestamp) {

1129:         return uint32(block.timestamp);

1134:     function _currCycleStart() private view returns (uint32 timestamp) {

1135:         uint32 currTimestamp = _currTimestamp();

```

```solidity
File: DripsHub.sol

19:     bytes value;

58:     uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;

64:     uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;

84:         uint32 indexed driverId, address indexed oldDriverAddr, address indexed newDriverAddr

97:         uint32 nextDriverId;

119:         uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);

145:     function driverAddress(uint32 driverId) public view returns (address driverAddr) {

152:     function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {

160:     function nextDriverId() public view returns (uint32 driverId) {

169:     function cycleSecs() public view returns (uint32 cycleSecs_) {

199:         returns (uint32 cycles)

216:     function receiveDripsResult(uint256 userId, IERC20 erc20, uint32 maxCycles)

219:         returns (uint128 receivableAmt)

238:     function receiveDrips(uint256 userId, IERC20 erc20, uint32 maxCycles)

241:         returns (uint128 receivedAmt)

275:         DripsHistory[] memory dripsHistory

302:         DripsHistory[] memory dripsHistory

328:     function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)

331:         returns (uint128 collectableAmt, uint128 splitAmt)

355:     function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)

358:         returns (uint128 collectableAmt, uint128 splitAmt)

372:     function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

390:         returns (uint128 amt)

409:     function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)

438:             uint32 updateTime,

439:             uint128 balance,

440:             uint32 maxEnd

463:         DripsReceiver[] memory receivers,

464:         uint32 timestamp

511:         uint256 userId,

513:         DripsReceiver[] memory currReceivers,

515:         DripsReceiver[] memory newReceivers,

517:         uint32 maxEndHint1,

518:         uint32 maxEndHint2

521:             _increaseTotalBalance(erc20, uint128(balanceDelta));

533:             erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));

535:             _decreaseTotalBalance(erc20, uint128(-realBalanceDelta));

536:             erc20.safeTransfer(msg.sender, uint128(-realBalanceDelta));

546:     function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {

560:         uint32 updateTime,

561:         uint32 maxEnd

576:     function setSplits(uint256 userId, SplitsReceiver[] memory receivers)

595:     function hashSplits(SplitsReceiver[] memory receivers)

598:         returns (bytes32 receiversHash)

608:     function emitUserMetadata(uint256 userId, UserMetadata[] calldata userMetadata)

629:     function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {

635:     function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {

643:         return uint160(address(erc20));

```

```solidity
File: ImmutableSplitsDriver.sol

15:     uint32 public immutable driverId;

17:     uint32 public immutable totalSplitsWeight;

53:     function createSplits(SplitsReceiver[] calldata receivers, UserMetadata[] calldata userMetadata)

```

```solidity
File: Managed.sol

112:     function allPausers() public view returns (address[] memory pausersList) {

```

```solidity
File: NFTDriver.sol

25:     uint32 public immutable driverId;

62:     function mint(address to, UserMetadata[] calldata userMetadata)

79:     function safeMint(address to, UserMetadata[] calldata userMetadata)

113:         returns (uint128 amt)

133:     function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)

189:         DripsReceiver[] calldata currReceivers,

191:         DripsReceiver[] calldata newReceivers,

193:         uint32 maxEndHint1,

194:         uint32 maxEndHint2,

198:             _transferFromCaller(erc20, uint128(balanceDelta));

204:             erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));

220:     function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)

235:     function emitUserMetadata(uint256 tokenId, UserMetadata[] calldata userMetadata)

285:     function _transferFromCaller(IERC20 erc20, uint128 amt) internal {

```

```solidity
File: Splits.sol

11:     uint32 weight;

22:     uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;

42:         uint256 indexed userId, uint256 indexed receiver, uint256 indexed assetId, uint128 amt

57:         uint256 indexed userId, uint256 indexed receiver, uint256 indexed assetId, uint128 amt

88:         uint128 splittable;

90:         uint128 collectable;

98:     function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal {

117:     function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)

120:         returns (uint128 collectableAmt, uint128 splitAmt)

126:         uint32 splitsWeight = 0;

130:         splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);

143:     function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)

145:         returns (uint128 collectableAmt, uint128 splitAmt)

157:         uint32 splitsWeight = 0;

160:             uint128 currSplitAmt =

161:                 uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);

199:     function _give(uint256 userId, uint256 receiver, uint256 assetId, uint128 amt) internal {

212:     function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {

226:     function _assertSplitsValid(SplitsReceiver[] memory receivers, bytes32 receiversHash) private {

228:         uint64 totalWeight = 0;

233:             uint32 weight = receiver.weight;

250:     function _assertCurrSplits(uint256 userId, SplitsReceiver[] memory currReceivers)

270:     function _hashSplits(SplitsReceiver[] memory receivers)
```

### [GAS-11] `require()` or `revert()` statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas\*) in a function that may ultimately revert in the unhappy case.

#### **Proof Of Concept**

```solidity
File: Drips.sol

461:         require(historyHash == finalHistoryHash, "Invalid drips history");

```

### [GAS-12] Functions not used internally could be marked external

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

##### **Proof Of Concept**

```solidity
File: AddressDriver.sol

60:     function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {

76:     function give(uint256 receiver, IERC20 erc20, uint128 amt) public whenNotPaused {

158:     function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {

166:     function emitUserMetadata(UserMetadata[] calldata userMetadata) public whenNotPaused {

```

```solidity
File: Caller.sol

106:     function authorize(address user) public {

114:     function unauthorize(address user) public {

132:     function allAuthorized(address sender) public view returns (address[] memory authorized) {

193:     function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {

```

```solidity
File: DripsHub.sol

134:     function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {

152:     function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {

160:     function nextDriverId() public view returns (uint32 driverId) {

169:     function cycleSecs() public view returns (uint32 cycleSecs_) {

181:     function totalBalance(IERC20 erc20) public view returns (uint256 balance) {

317:     function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

372:     function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

546:     function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {

587:     function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {

```

```solidity
File: Managed.sol

84:     function changeAdmin(address newAdmin) public onlyAdmin {

90:     function grantPauser(address pauser) public onlyAdmin {

97:     function revokePauser(address pauser) public onlyAdmin {

112:     function allPausers() public view returns (address[] memory pausersList) {

122:     function pause() public onlyAdminOrPauser whenNotPaused {

128:     function unpause() public onlyAdminOrPauser whenPaused {

```

```solidity
File: NFTDriver.sol

244:     function burn(uint256 tokenId) public override whenNotPaused {

249:     function approve(address to, uint256 tokenId) public override whenNotPaused {

272:     function setApprovalForAll(address operator, bool approved) public override whenNotPaused {

```

### [GAS-13] Using storage instead of memory for structs/arrays saves gas

#### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

##### **Proof Of Concept**

```solidity
File: Drips.sol

792:     function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver)

815:     function _getConfig(uint256[] memory configs, uint256 idx)

874:         DripsReceiver[] memory currReceivers,

877:         DripsReceiver[] memory newReceivers,

986:         DripsReceiver memory receiver,
```

### [GAS-14] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

#### **Context:**

All Contracts apart from Caller.sol and Managed.sol

### [GAS-15] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

Do not use return at the end of the function:

#### **Proof Of Concept**

```solidity
File: Drips.sol

60:      function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)

```

```solidity
File: Splits.sol

270:     function _hashSplits(SplitsReceiver[] memory receivers)

283:     function _splitsStorage() private view returns (SplitsStorage storage splitsStorage) {

```

### [GAS-16] NOT USING THE NAMED RETURN VARIABLES IN THE FUNCTION IS CONFUSING

#### **Proof Of Concept**

```solidity
File: Drips.sol

63:         returns (DripsConfig)

73:     function dripId(DripsConfig config) internal pure returns (uint32) {

78:     function amtPerSec(DripsConfig config) internal pure returns (uint160) {

83:     function start(DripsConfig config) internal pure returns (uint32) {

88:     function duration(DripsConfig config) internal pure returns (uint32) {

94:     function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {

1064:         returns (bool)

```

```solidity
File: Managed.sol

78:     function admin() public view returns (address) {
```

```solidity
File: NFTDriver.sol

294:     function _msgSender() internal view override(Context, ERC2771Context) returns (address) {

300:     function _msgData() internal view override(Context, ERC2771Context) returns (bytes calldata) {

```

#### Recommende Mitigation Steps

Consider adopting a consistent approach to return values throughout the codebase by adding everywhere named return variables.
