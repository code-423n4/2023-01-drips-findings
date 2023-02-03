# Gas Optimizations Report
[Excluded findings](#excluded-findings) are below the automated findings.


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Cache array length outside of loop | 13 |
| [GAS-2](#GAS-2) | State variables should be cached in stack variables rather than re-reading them from storage | 2 |
| [GAS-3](#GAS-3) | Use calldata instead of memory for function arguments that do not get mutated | 14 |
| [GAS-4](#GAS-4) | Use Custom Errors | 27 |
| [GAS-5](#GAS-5) | Don't initialize variables with default value | 21 |
| [GAS-6](#GAS-6) | Long revert strings | 3 |
| [GAS-7](#GAS-7) | Functions guaranteed to revert when called by normal users can be marked `payable` | 6 |
| [GAS-8](#GAS-8) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 24 |
| [GAS-9](#GAS-9) | Using `private` rather than `public` for constants, saves gas | 7 |
| [GAS-10](#GAS-10) | Use shift Right/Left instead of division/multiplication if possible | 2 |
| [GAS-11](#GAS-11) | Use != 0 instead of > 0 for unsigned integer comparison | 11 |
| [GAS-12](#GAS-12) | `internal` functions not called by the contract should be removed | 18 |
### <a name="GAS-1"></a>[GAS-1] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (13)*:
```solidity
File: Caller.sol

196:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: Drips.sol

422:         for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

450:         for (uint256 i = 0; i < dripsHistory.length; i++) {

479:         for (uint256 idxCap = receivers.length; idx < idxCap;) {

490:         for (; idx < receivers.length; idx++) {

563:         for (uint256 i = 0; i < receivers.length; i++) {

664:             for (uint256 i = 0; i < newReceivers.length; i++) {

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

### <a name="GAS-2"></a>[GAS-2] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (2)*:
```solidity
File: AddressDriver.sol

174:             erc20.safeApprove(address(dripsHub), type(uint256).max);

```

```solidity
File: NFTDriver.sol

289:             erc20.safeApprove(address(dripsHub), type(uint256).max);

```

### <a name="GAS-3"></a>[GAS-3] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (14)*:
```solidity
File: Caller.sol

144:     function callAs(address sender, address to, bytes memory data)

167:         bytes memory data,

193:     function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {

```

```solidity
File: DripsHub.sol

275:         DripsHistory[] memory dripsHistory

302:         DripsHistory[] memory dripsHistory

328:     function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)

355:     function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)

463:         DripsReceiver[] memory receivers,

513:         DripsReceiver[] memory currReceivers,

515:         DripsReceiver[] memory newReceivers,

546:     function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {

576:     function setSplits(uint256 userId, SplitsReceiver[] memory receivers)

595:     function hashSplits(SplitsReceiver[] memory receivers)

```

```solidity
File: NFTDriver.sol

263:     function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data)

```

### <a name="GAS-4"></a>[GAS-4] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (27)*:
```solidity
File: Caller.sol

108:         require(_authorized[sender].add(user), "Address already is authorized");

116:         require(_authorized[sender].remove(user), "Address is not authorized");

149:         require(isAuthorized(sender, _msgSender()), "Not authorized");

173:         require(block.timestamp <= deadline, "Execution deadline expired");

181:         require(signer == sender, "Invalid signature");

```

```solidity
File: Drips.sol

220:         require(cycleSecs > 1, "Cycle length too low");

454:                 require(dripsHash == 0, "Drips history entry with hash and receivers");

461:         require(historyHash == finalHistoryHash, "Invalid drips history");

540:         require(timestamp >= state.updateTime, "Timestamp before last drips update");

541:         require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list");

622:         require(_hashDrips(currReceivers) == state.dripsHash, "Invalid current drips list");

775:             require(receivers.length <= _MAX_DRIPS_RECEIVERS, "Too many drips receivers");

780:                     require(_isOrdered(receivers[i - 1], receiver), "Receivers not sorted");

798:         require(amtPerSec != 0, "Drips receiver amtPerSec is zero");

```

```solidity
File: DripsHub.sol

125:         require(driverAddress(driverId) == msg.sender, "Callable only by the driver");

631:         require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");

```

```solidity
File: ImmutableSplitsDriver.sol

64:         require(weightSum == totalSplitsWeight, "Invalid total receivers weight");

```

```solidity
File: Managed.sol

47:         require(admin() == msg.sender, "Caller is not the admin");

61:         require(!paused(), "Contract paused");

67:         require(paused(), "Contract not paused");

91:         require(_managedStorage().pausers.add(pauser), "Address already is a pauser");

98:         require(_managedStorage().pausers.remove(pauser), "Address is not a pauser");

```

```solidity
File: Splits.sol

227:         require(receivers.length <= _MAX_SPLITS_RECEIVERS, "Too many splits receivers");

234:             require(weight != 0, "Splits receiver weight is zero");

238:                 require(prevUserId != userId, "Duplicate splits receivers");

239:                 require(prevUserId < userId, "Splits receivers not sorted by user ID");

244:         require(totalWeight <= _TOTAL_SPLITS_WEIGHT, "Splits weights sum too high");

```

### <a name="GAS-5"></a>[GAS-5] Don't initialize variables with default value

*Instances (21)*:
```solidity
File: Caller.sol

196:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: Drips.sol

358:         for (uint256 i = 0; i < squeezedNum; i++) {

450:         for (uint256 i = 0; i < dripsHistory.length; i++) {

478:         uint256 idx = 0;

489:         uint256 amt = 0;

563:         for (uint256 i = 0; i < receivers.length; i++) {

664:             for (uint256 i = 0; i < newReceivers.length; i++) {

744:             uint256 spent = 0;

745:             for (uint256 i = 0; i < configsLen; i++) {

777:             for (uint256 i = 0; i < receivers.length; i++) {

880:         uint256 currIdx = 0;

881:         uint256 newIdx = 0;

```

```solidity
File: DripsHub.sol

613:         for (uint256 i = 0; i < userMetadata.length; i++) {

```

```solidity
File: ImmutableSplitsDriver.sol

60:         uint256 weightSum = 0;

61:         for (uint256 i = 0; i < receivers.length; i++) {

```

```solidity
File: Splits.sol

126:         uint32 splitsWeight = 0;

127:         for (uint256 i = 0; i < currReceivers.length; i++) {

157:         uint32 splitsWeight = 0;

158:         for (uint256 i = 0; i < currReceivers.length; i++) {

228:         uint64 totalWeight = 0;

231:         for (uint256 i = 0; i < receivers.length; i++) {

```

### <a name="GAS-6"></a>[GAS-6] Long revert strings

*Instances (3)*:
```solidity
File: Drips.sol

454:                 require(dripsHash == 0, "Drips history entry with hash and receivers");

540:         require(timestamp >= state.updateTime, "Timestamp before last drips update");

```

```solidity
File: Splits.sol

239:                 require(prevUserId < userId, "Splits receivers not sorted by user ID");

```

### <a name="GAS-7"></a>[GAS-7] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (6)*:
```solidity
File: Managed.sol

84:     function changeAdmin(address newAdmin) public onlyAdmin {

90:     function grantPauser(address pauser) public onlyAdmin {

97:     function revokePauser(address pauser) public onlyAdmin {

122:     function pause() public onlyAdminOrPauser whenNotPaused {

128:     function unpause() public onlyAdminOrPauser whenPaused {

151:     function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {

```

### <a name="GAS-8"></a>[GAS-8] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (24)*:
```solidity
File: Caller.sol

174:         uint256 currNonce = nonce[sender]++;

196:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: Drips.sol

247:             for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

287:         for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

358:         for (uint256 i = 0; i < squeezedNum; i++) {

422:         for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

428:                     squeezedRevIdxs[squeezedNum++] = i;

450:         for (uint256 i = 0; i < dripsHistory.length; i++) {

490:         for (; idx < receivers.length; idx++) {

563:         for (uint256 i = 0; i < receivers.length; i++) {

655:             state.currCycleConfigs++;

664:             for (uint256 i = 0; i < newReceivers.length; i++) {

745:             for (uint256 i = 0; i < configsLen; i++) {

777:             for (uint256 i = 0; i < receivers.length; i++) {

959:                 currIdx++;

962:                 newIdx++;

```

```solidity
File: DripsHub.sol

136:         driverId = dripsHubStorage.nextDriverId++;

613:         for (uint256 i = 0; i < userMetadata.length; i++) {

```

```solidity
File: ImmutableSplitsDriver.sol

59:         StorageSlot.getUint256Slot(_counterSlot).value++;

61:         for (uint256 i = 0; i < receivers.length; i++) {

```

```solidity
File: NFTDriver.sol

93:         StorageSlot.getUint256Slot(_mintedTokensSlot).value++;

```

```solidity
File: Splits.sol

127:         for (uint256 i = 0; i < currReceivers.length; i++) {

158:         for (uint256 i = 0; i < currReceivers.length; i++) {

231:         for (uint256 i = 0; i < receivers.length; i++) {

```

### <a name="GAS-9"></a>[GAS-9] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (7)*:
```solidity
File: DripsHub.sol

56:     uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;

58:     uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;

60:     uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;

62:     uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;

64:     uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;

67:     uint256 public constant DRIVER_ID_OFFSET = 224;

70:     uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;

```

### <a name="GAS-10"></a>[GAS-10] Use shift Right/Left instead of division/multiplication if possible

*Instances (2)*:
```solidity
File: Drips.sol

480:             uint256 idxMid = (idx + idxCap) / 2;

717:                 uint256 end = (enoughEnd + notEnoughEnd) / 2;

```

### <a name="GAS-11"></a>[GAS-11] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (11)*:
```solidity
File: AddressDriver.sol

132:         if (balanceDelta > 0) {

```

```solidity
File: Drips.sol

779:                 if (i > 0) {

```

```solidity
File: DripsHub.sol

245:         if (receivedAmt > 0) {

279:         if (amt > 0) {

520:         if (balanceDelta > 0) {

532:         if (realBalanceDelta > 0) {

```

```solidity
File: ImmutableSplitsDriver.sol

67:         if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);

```

```solidity
File: NFTDriver.sol

69:         if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);

86:         if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);

197:         if (balanceDelta > 0) {

```

```solidity
File: Splits.sol

237:             if (i > 0) {

```

### <a name="GAS-12"></a>[GAS-12] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (18)*:
```solidity
File: Drips.sol

60:     function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)

73:     function dripId(DripsConfig config) internal pure returns (uint32) {

78:     function amtPerSec(DripsConfig config) internal pure returns (uint160) {

83:     function start(DripsConfig config) internal pure returns (uint32) {

88:     function duration(DripsConfig config) internal pure returns (uint32) {

94:     function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {

233:     function _receiveDrips(uint256 userId, uint256 assetId, uint32 maxCycles)

300:     function _receivableDripsCycles(uint256 userId, uint256 assetId)

342:     function _squeezeDrips(

508:     function _dripsState(uint256 userId, uint256 assetId)

610:     function _setDrips(

```

```solidity
File: Splits.sol

106:     function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {

117:     function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)

143:     function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)

176:     function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {

185:     function _collect(uint256 userId, uint256 assetId) internal returns (uint128 amt) {

199:     function _give(uint256 userId, uint256 receiver, uint256 assetId, uint128 amt) internal {

212:     function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {

```


---
## Excluded Findings
| No. | Issue |
| --- | --- |
| 1 | [Reduce the size of error messages (Long revert Strings)](#G-01-reduce-the-size-of-error-messages-long-revert-strings)|
| 2 | [Use Custom Errors rather than revert() / require() strings to save deployment gas [68 gas per instance]](#G-02-use-custom-errors-rather-than-revert--require-strings-to-save-deployment-gas-68-gas-per-instance)|
| 3 | [Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#G-03-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead)|
| 4 | [++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#G-04-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops)|
| 5 | [Optimize names to save gas [22 gas per instance]](#G-05-optimize-names-to-save-gas-22-gas-per-instance)|
| 6 | [Setting the `constructor` to `payable` [~13 gas per instance]](#G-06-setting-the-constructor-to-payable-13-gas-per-instance)|
| 7 | [Comparison operators](#G-07-comparison-operators)|
| 8 | [Use a more recent version of solidity](#G-08-use-a-more-recent-version-of-solidity)|
| 9 | [`<array>.length` should not be looked up in every loop of a for-loop](#G-09-arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)|
| 10 | [Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate](#G-10-multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate)|

---
### [G-01] Reduce the size of error messages (Long revert Strings)

---
**Description:**

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition has been met. 

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

**Recommendation:**

Shorten the revert strings to fit in 32 bytes. 

Or in contracts using solc version 0.8.4 or greater use the Custom Errors feature.

**Lines of Code:** 

- [Drips.sol#L454](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L454)
- [Drips.sol#L540](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L540)
- [Managed.sol#L54](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L54)
- [NFTDriver.sol#L43](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L43)
- [Splits.sol#L239](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L239)

---
### [G-02] Use Custom Errors rather than revert() / require() strings to save deployment gas [68 gas per instance]

---
**Description:**

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hitby avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas.

<https://blog.soliditylang.org/2021/04/21/custom-errors/>

**Recommendation:**

Use the Custom Errors feature.

**Lines of Code:** 

- [Caller.sol#L108](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L108)
- [Caller.sol#L116](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L116)
- [Caller.sol#L149](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L149)
- [Caller.sol#L173](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L173)
- [Caller.sol#L181](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L181)
- [Drips.sol#L220](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L220)
- [Drips.sol#L454](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L454)
- [Drips.sol#L461](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L461)
- [Drips.sol#L540](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L540)
- [Drips.sol#L541](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L541)
- [Drips.sol#L622](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L622)
- [Drips.sol#L775](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L775)
- [Drips.sol#L780](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L780)
- [Drips.sol#L798](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L798)
- [DripsHub.sol#L125](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L125)
- [DripsHub.sol#L631](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L631)
- [ImmutableSplitsDriver.sol#L64](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L64)
- [Managed.sol#L47](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L47)
- [Managed.sol#L53-L55](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L53-L55)
- [Managed.sol#L61](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L61)
- [Managed.sol#L67](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L67)
- [Managed.sol#L91](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L91)
- [Managed.sol#L98](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L98)
- [NFTDriver.sol#L41-L44](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L41-L44)
- [Splits.sol#L227](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L227)
- [Splits.sol#L234](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L234)
- [Splits.sol#L238](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L238)
- [Splits.sol#L239](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L239)
- [Splits.sol#L244](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L244)
- [Splits.sol#L254-L256](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L254-L256)

---
### [G-03] Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

---
**Description:**

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so.

**Recommendation:**

Use a larger size then downcast where needed

**Lines of Code:** 

- [AddressDriver.sol#L25](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L25)
- [AddressDriver.sol#L30](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L30)
- [AddressDriver.sol#L60](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L60)
- [AddressDriver.sol#L76](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L76)
- [AddressDriver.sol#L125](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L125)
- [AddressDriver.sol#L128](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L128)
- [AddressDriver.sol#L129](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L129)
- [AddressDriver.sol#L131](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L131)
- [AddressDriver.sol#L170](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L170)
- [Drips.sol#L28](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L28)
- [Drips.sol#L30](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L30)
- [Drips.sol#L60](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L60)
- [Drips.sol#L60](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L60)
- [Drips.sol#L60](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L60)
- [Drips.sol#L60](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L60)
- [Drips.sol#L108](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L108)
- [Drips.sol#L117](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L117)
- [Drips.sol#L135](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L135)
- [Drips.sol#L136](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L136)
- [Drips.sol#L153](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L153)
- [Drips.sol#L153](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L153)
- [Drips.sol#L169](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L169)
- [Drips.sol#L189](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L189)
- [Drips.sol#L191](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L191)
- [Drips.sol#L193](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L193)
- [Drips.sol#L195](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L195)
- [Drips.sol#L197](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L197)
- [Drips.sol#L203](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L203)
- [Drips.sol#L208](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L208)
- [Drips.sol#L210](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L210)
- [Drips.sol#L219](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L219)
- [Drips.sol#L233](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L233)
- [Drips.sol#L235](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L235)
- [Drips.sol#L237](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L237)
- [Drips.sol#L238](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L238)
- [Drips.sol#L239](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L239)
- [Drips.sol#L240](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L240)
- [Drips.sol#L246](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L246)
- [Drips.sol#L247](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L247)
- [Drips.sol#L270](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L270)
- [Drips.sol#L274](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L274)
- [Drips.sol#L275](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L275)
- [Drips.sol#L276](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L276)
- [Drips.sol#L277](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L277)
- [Drips.sol#L278](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L278)
- [Drips.sol#L287](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L287)
- [Drips.sol#L303](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L303)
- [Drips.sol#L305](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L305)
- [Drips.sol#L305](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L305)
- [Drips.sol#L317](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L317)
- [Drips.sol#L317](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L317)
- [Drips.sol#L348](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L348)
- [Drips.sol#L365](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L365)
- [Drips.sol#L402](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L402)
- [Drips.sol#L421](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L421)
- [Drips.sol#L425](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L425)
- [Drips.sol#L473](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L473)
- [Drips.sol#L474](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L474)
- [Drips.sol#L475](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L475)
- [Drips.sol#L487](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L487)
- [Drips.sol#L488](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L488)
- [Drips.sol#L493](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L493)
- [Drips.sol#L493](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L493)
- [Drips.sol#L514](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L514)
- [Drips.sol#L515](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L515)
- [Drips.sol#L516](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L516)
- [Drips.sol#L537](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L537)
- [Drips.sol#L538](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L538)
- [Drips.sol#L556](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L556)
- [Drips.sol#L557](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L557)
- [Drips.sol#L558](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L558)
- [Drips.sol#L560](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L560)
- [Drips.sol#L561](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L561)
- [Drips.sol#L565](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L565)
- [Drips.sol#L565](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L565)
- [Drips.sol#L614](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L614)
- [Drips.sol#L617](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L617)
- [Drips.sol#L618](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L618)
- [Drips.sol#L619](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L619)
- [Drips.sol#L623](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L623)
- [Drips.sol#L624](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L624)
- [Drips.sol#L625](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L625)
- [Drips.sol#L627](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L627)
- [Drips.sol#L628](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L628)
- [Drips.sol#L681](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L681)
- [Drips.sol#L683](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L683)
- [Drips.sol#L684](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L684)
- [Drips.sol#L685](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L685)
- [Drips.sol#L855](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L855)
- [Drips.sol#L856](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L856)
- [Drips.sol#L875](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L875)
- [Drips.sol#L876](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L876)
- [Drips.sol#L878](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L878)
- [Drips.sol#L912](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L912)
- [Drips.sol#L912](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L912)
- [Drips.sol#L914](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L914)
- [Drips.sol#L914](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L914)
- [Drips.sol#L923](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L923)
- [Drips.sol#L924](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L924)
- [Drips.sol#L936](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L936)
- [Drips.sol#L936](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L936)
- [Drips.sol#L944](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L944)
- [Drips.sol#L944](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L944)
- [Drips.sol#L949](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L949)
- [Drips.sol#L970](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L970)
- [Drips.sol#L970](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L970)
- [Drips.sol#L973](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L973)
- [Drips.sol#L973](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L973)
- [Drips.sol#L987](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L987)
- [Drips.sol#L988](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L988)
- [Drips.sol#L989](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L989)
- [Drips.sol#L990](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L990)
- [Drips.sol#L991](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L991)
- [Drips.sol#L991](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L991)
- [Drips.sol#L997](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L997)
- [Drips.sol#L1020](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1020)
- [Drips.sol#L1020](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1020)
- [Drips.sol#L1027](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1027)
- [Drips.sol#L1037](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1037)
- [Drips.sol#L1120](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1120)
- [Drips.sol#L1120](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1120)
- [Drips.sol#L1128](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1128)
- [Drips.sol#L1134](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1134)
- [Drips.sol#L1135](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1135)
- [DripsHub.sol#L58](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L58)
- [DripsHub.sol#L64](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L64)
- [DripsHub.sol#L77](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L77)
- [DripsHub.sol#L84](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L84)
- [DripsHub.sol#L97](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L97)
- [DripsHub.sol#L99](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L99)
- [DripsHub.sol#L109](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L109)
- [DripsHub.sol#L119](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L119)
- [DripsHub.sol#L124](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L124)
- [DripsHub.sol#L134](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L134)
- [DripsHub.sol#L145](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L145)
- [DripsHub.sol#L152](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L152)
- [DripsHub.sol#L160](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L160)
- [DripsHub.sol#L169](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L169)
- [DripsHub.sol#L199](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L199)
- [DripsHub.sol#L216](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L216)
- [DripsHub.sol#L219](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L219)
- [DripsHub.sol#L238](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L238)
- [DripsHub.sol#L241](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L241)
- [DripsHub.sol#L276](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L276)
- [DripsHub.sol#L303](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L303)
- [DripsHub.sol#L317](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L317)
- [DripsHub.sol#L328](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L328)
- [DripsHub.sol#L331](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L331)
- [DripsHub.sol#L331](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L331)
- [DripsHub.sol#L358](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L358)
- [DripsHub.sol#L358](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L358)
- [DripsHub.sol#L372](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L372)
- [DripsHub.sol#L390](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L390)
- [DripsHub.sol#L409](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L409)
- [DripsHub.sol#L438](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L438)
- [DripsHub.sol#L439](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L439)
- [DripsHub.sol#L440](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L440)
- [DripsHub.sol#L464](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L464)
- [DripsHub.sol#L465](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L465)
- [DripsHub.sol#L514](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L514)
- [DripsHub.sol#L517](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L517)
- [DripsHub.sol#L518](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L518)
- [DripsHub.sol#L519](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L519)
- [DripsHub.sol#L560](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L560)
- [DripsHub.sol#L561](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L561)
- [DripsHub.sol#L629](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L629)
- [DripsHub.sol#L635](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L635)
- [ImmutableSplitsDriver.sol#L15](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L15)
- [ImmutableSplitsDriver.sol#L17](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L17)
- [ImmutableSplitsDriver.sol#L28](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L28)
- [NFTDriver.sol#L25](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L25)
- [NFTDriver.sol#L32](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L32)
- [NFTDriver.sol#L113](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L113)
- [NFTDriver.sol#L133](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L133)
- [NFTDriver.sol#L190](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L190)
- [NFTDriver.sol#L193](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L193)
- [NFTDriver.sol#L194](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L194)
- [NFTDriver.sol#L196](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L196)
- [NFTDriver.sol#L285](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L285)
- [Splits.sol#L11](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L11)
- [Splits.sol#L22](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L22)
- [Splits.sol#L33](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L33)
- [Splits.sol#L42](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L42)
- [Splits.sol#L49](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L49)
- [Splits.sol#L57](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L57)
- [Splits.sol#L71](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L71)
- [Splits.sol#L88](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L88)
- [Splits.sol#L90](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L90)
- [Splits.sol#L98](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L98)
- [Splits.sol#L106](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L106)
- [Splits.sol#L117](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L117)
- [Splits.sol#L120](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L120)
- [Splits.sol#L120](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L120)
- [Splits.sol#L126](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L126)
- [Splits.sol#L145](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L145)
- [Splits.sol#L145](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L145)
- [Splits.sol#L157](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L157)
- [Splits.sol#L160](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L160)
- [Splits.sol#L176](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L176)
- [Splits.sol#L185](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L185)
- [Splits.sol#L199](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L199)
- [Splits.sol#L228](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L228)
- [Splits.sol#L233](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L233)

---
### [G-04] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

---
**Description:**

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

**Recommendation:**

Using Solidity's unchecked block saves the overflow checks.

**Proof Of Concept:**

<https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g011---unnecessary-checked-arithmetic-in-for-loop>

**Lines of Code:** 

- [Caller.sol#L196](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L196)
- [Drips.sol#L358](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L358)
- [Drips.sol#L422](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L422)
- [Drips.sol#L450](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L450)
- [Drips.sol#L563](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L563)
- [Drips.sol#L664](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L664)
- [Drips.sol#L745](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L745)
- [Drips.sol#L777](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L777)
- [DripsHub.sol#L613](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L613)
- [ImmutableSplitsDriver.sol#L61](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L61)
- [Splits.sol#L127](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L127)
- [Splits.sol#L158](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L158)
- [Splits.sol#L231](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L231)

---
### [G-05] Optimize names to save gas [22 gas per instance]

---
**Description:**

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Recommendation:**

Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas. For example, the function IDs in the L1GraphTokenGateway.sol contract will be the most used; A lower method ID may be given.

**Proof Of Concept:**

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

**Lines of Code:** 

- [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol)
- [Caller.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol)
- [Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol)
- [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol)
- [ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol)
- [Managed.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol)
- [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol)
- [Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol)

---
### [G-06] Setting the `constructor` to `payable` [~13 gas per instance]

---
**Lines of Code:** 

- [AddressDriver.sol#L30](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L30)
- [Drips.sol#L219](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L219)
- [DripsHub.sol#L109](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L109)
- [ImmutableSplitsDriver.sol#L28](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L28)
- [Managed.sol#L73](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L73)
- [Managed.sol#L158](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L158)
- [NFTDriver.sol#L32](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L32)
- [Splits.sol#L94](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L94)

---
### [G-07] Comparison operators

---
**Description:**

In the EVM, there is no opcode for `>=` or `<=`. When using greater than or equal, two operations are performed: `>` and `=`. Using strict comparison operators hence saves gas.

**Recommendation:**

 Replace `<=` with `<`, and `>=` with `>`. Do not forget to increment/decrement the compared variable.

**Lines of Code:** 

- [Caller.sol#L173](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L173)
- [Drips.sol#L416](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L416)
- [Drips.sol#L422](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L422)
- [Drips.sol#L422](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L422)
- [Drips.sol#L540](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L540)
- [Drips.sol#L748](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L748)
- [Drips.sol#L775](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L775)
- [DripsHub.sol#L631](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L631)
- [Splits.sol#L227](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L227)
- [Splits.sol#L244](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L244)

---
### [G-08] Use a more recent version of solidity

---
**Description:**

Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value.

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

**Lines of Code:** 

- [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol)
- [Caller.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol)
- [Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol)
- [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol)
- [ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol)
- [Managed.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol)
- [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol)
- [Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol)

---
### [G-09] `<array>.length` should not be looked up in every loop of a for-loop

---
**Description:**

The overheads outlined below are *PER LOOP*, excluding the first loop 

- storage arrays incur a Gwarmaccess (100 gas)

- memory arrays use `MLOAD` (3 gas)

- calldata arrays use `CALLDATALOAD` (3 gas)

Caching the length changes each of these to a `DUP<N>` (3 gas), and gets rid of the extra DUP<N> needed to store the stack offset



**Lines of Code:** 

- [Caller.sol#L196](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L196)
- [Drips.sol#L422](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L422)
- [Drips.sol#L450](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L450)
- [Drips.sol#L479](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L479)
- [Drips.sol#L490](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L490)
- [Drips.sol#L563](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L563)
- [Drips.sol#L664](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L664)
- [Drips.sol#L777](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L777)
- [DripsHub.sol#L613](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L613)
- [ImmutableSplitsDriver.sol#L61](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L61)
- [Splits.sol#L127](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L127)
- [Splits.sol#L158](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L158)
- [Splits.sol#L231](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L231)
---
### [G-10] Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate

---
**Description:**

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot

**Lines of Code:** 

- [Caller.sol#L88-L106](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L88-L106)

