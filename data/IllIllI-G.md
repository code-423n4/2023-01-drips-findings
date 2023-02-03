## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Using `storage` instead of `memory` for structs/arrays saves gas | 1 |  4200 |
| [G&#x2011;02] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 14 |  840 |
| [G&#x2011;03] | `require()`/`revert()` strings longer than 32 bytes cost extra gas | 2 |  - |
| [G&#x2011;04] | Optimize names to save gas | 6 |  132 |
| [G&#x2011;05] | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 34 |  - |
| [G&#x2011;06] | Using `private` rather than `public` for constants, saves gas | 4 |  - |
| [G&#x2011;07] | Use custom errors rather than `revert()`/`require()` strings to save gas | 3 |  - |
| [G&#x2011;08] | Functions guaranteed to revert when called by normal users can be marked `payable` | 10 |  210 |
| [G&#x2011;09] | Constructors can be marked `payable` | 8 |  168 |

Total: 82 instances over 9 issues with **5550 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Using `storage` instead of `memory` for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

*There is 1 instance of this issue:*

```solidity
File: src/Drips.sol

491:              DripsReceiver memory receiver = receivers[idx];

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L491

### [G&#x2011;02]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**

*There are 14 instances of this issue:*

```solidity
File: src/Caller.sol

196:          for (uint256 i = 0; i < calls.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L196

```solidity
File: src/DripsHub.sol

613:          for (uint256 i = 0; i < userMetadata.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L613

```solidity
File: src/Drips.sol

247:              for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

287:          for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

358:          for (uint256 i = 0; i < squeezedNum; i++) {

422:          for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

450:          for (uint256 i = 0; i < dripsHistory.length; i++) {

490:          for (; idx < receivers.length; idx++) {

563:          for (uint256 i = 0; i < receivers.length; i++) {

664:              for (uint256 i = 0; i < newReceivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L247

```solidity
File: src/ImmutableSplitsDriver.sol

61:           for (uint256 i = 0; i < receivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L61

```solidity
File: src/Splits.sol

127:          for (uint256 i = 0; i < currReceivers.length; i++) {

158:          for (uint256 i = 0; i < currReceivers.length; i++) {

231:          for (uint256 i = 0; i < receivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L127

### [G&#x2011;03]  `require()`/`revert()` strings longer than 32 bytes cost extra gas
Each extra memory word of bytes past the original 32 [incurs an MSTORE](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings) which costs **3 gas**

*There are 2 instances of this issue:*

```solidity
File: src/Managed.sol

53            require(
54                admin() == msg.sender || isPauser(msg.sender), "Caller is not the admin or a pauser"
55:           );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L53-L55

```solidity
File: src/NFTDriver.sol

41            require(
42                _isApprovedOrOwner(_msgSender(), tokenId),
43                "ERC721: caller is not token owner or approved"
44:           );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L41-L44

### [G&#x2011;04]  Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

*There are 6 instances of this issue:*

```solidity
File: src/AddressDriver.sol

/// @audit calcUserId(), collect(), give(), setDrips(), setSplits(), emitUserMetadata()
19:   contract AddressDriver is Managed, ERC2771Context {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L19

```solidity
File: src/Caller.sol

/// @audit authorize(), unauthorize(), isAuthorized(), allAuthorized(), callAs(), callSigned(), callBatched()
81:   contract Caller is EIP712("Caller", "1"), ERC2771Context(address(this)) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L81

```solidity
File: src/DripsHub.sol

/// @audit registerDriver(), driverAddress(), updateDriverAddress(), nextDriverId(), cycleSecs(), totalBalance(), receivableDripsCycles(), receiveDripsResult(), receiveDrips(), squeezeDrips(), squeezeDripsResult(), splittable(), splitResult(), split(), collectable(), collect(), give(), dripsState(), balanceAt(), setDrips(), hashDrips(), hashDripsHistory(), setSplits(), splitsHash(), hashSplits(), emitUserMetadata()
53:   contract DripsHub is Managed, Drips, Splits {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L53

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit nextUserId(), createSplits()
11:   contract ImmutableSplitsDriver is Managed {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L11

```solidity
File: src/Managed.sol

/// @audit changeAdmin(), grantPauser(), revokePauser(), isPauser(), allPausers()
18:   abstract contract Managed is UUPSUpgradeable {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L18

```solidity
File: src/NFTDriver.sol

/// @audit nextTokenId(), mint(), safeMint(), collect(), give(), setDrips(), setSplits(), emitUserMetadata()
19:   contract NFTDriver is ERC721Burnable, ERC2771Context, Managed {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L19

### [G&#x2011;05]  Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
> When using elements that are smaller than 32 bytes, your contractâ€™s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

*There are 34 instances of this issue:*

```solidity
File: src/AddressDriver.sol

/// @audit uint128 amt
61:           amt = dripsHub.collect(callerUserId(), erc20);

/// @audit int128 realBalanceDelta
135           realBalanceDelta = dripsHub.setDrips(
136               callerUserId(),
137               erc20,
138               currReceivers,
139               balanceDelta,
140               newReceivers,
141               maxEndHint1,
142               maxEndHint2
143:          );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L61

```solidity
File: src/DripsHub.sol

/// @audit uint32 driverId
136:          driverId = dripsHubStorage.nextDriverId++;

/// @audit uint128 receivedAmt
244:          receivedAmt = Drips._receiveDrips(userId, assetId, maxCycles);

/// @audit uint128 amt
278:          amt = Drips._squeezeDrips(userId, assetId, senderId, historyHash, dripsHistory);

/// @audit uint128 amt
392:          amt = Splits._collect(userId, _assetId(erc20));

/// @audit int128 realBalanceDelta
523           realBalanceDelta = Drips._setDrips(
524               userId,
525               _assetId(erc20),
526               currReceivers,
527               balanceDelta,
528               newReceivers,
529               maxEndHint1,
530               maxEndHint2
531:          );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L136

```solidity
File: src/Drips.sol

/// @audit uint32 receivableCycles
283:              receivableCycles = toCycle - fromCycle - maxCycles;

/// @audit int128 amtPerCycle
288:              amtPerCycle += state.amtDeltas[cycle].thisCycle;

/// @audit uint128 receivedAmt
289:              receivedAmt += uint128(amtPerCycle);

/// @audit int128 amtPerCycle
290:              amtPerCycle += state.amtDeltas[cycle].nextCycle;

/// @audit uint32 fromCycle
319:          fromCycle = _dripsStorage().states[assetId][userId].nextReceivableCycle;

/// @audit uint32 toCycle
320:          toCycle = _cycleOf(_currTimestamp());

/// @audit uint32 squeezeStartCap
426:                  if (squeezeStartCap < _currCycleStart()) squeezeStartCap = _currCycleStart();

/// @audit uint128 amt
429:                      amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);

/// @audit uint32 squeezeEndCap
432:              squeezeEndCap = drips.updateTime;

/// @audit uint128 balance
572:              balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

/// @audit int128 realBalanceDelta
634:                  realBalanceDelta = -currBalance;

/// @audit uint128 newBalance
636:              newBalance = uint128(currBalance + realBalanceDelta);

/// @audit uint32 newMaxEnd
637:              newMaxEnd = _calcMaxEnd(newBalance, newReceivers, maxEndHint1, maxEndHint2);

/// @audit uint32 start
992:          start = receiver.config.start();

/// @audit uint40 end
1000:             end = maxEnd;

/// @audit uint40 end
1006:             end = endCap;

/// @audit uint40 end
1009:             end = start;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L283

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit uint32 totalSplitsWeight
31:           totalSplitsWeight = _dripsHub.TOTAL_SPLITS_WEIGHT();

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L31

```solidity
File: src/NFTDriver.sol

/// @audit uint128 amt
115:          amt = dripsHub.collect(tokenId, erc20);

/// @audit int128 realBalanceDelta
200           realBalanceDelta = dripsHub.setDrips(
201               tokenId, erc20, currReceivers, balanceDelta, newReceivers, maxEndHint1, maxEndHint2
202:          );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L115

```solidity
File: src/Splits.sol

/// @audit uint32 splitsWeight
128:              splitsWeight += currReceivers[i].weight;

/// @audit uint128 splitAmt
130:          splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);

/// @audit uint128 collectableAmt
131:          collectableAmt = amount - splitAmt;

/// @audit uint128 collectableAmt
151:          collectableAmt = balance.splittable;

/// @audit uint32 splitsWeight
159:              splitsWeight += currReceivers[i].weight;

/// @audit uint128 amt
187:          amt = balance.collectable;

/// @audit uint64 totalWeight
235:              totalWeight += weight;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L128

### [G&#x2011;06]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 4 instances of this issue:*

```solidity
File: src/AddressDriver.sol

25:       uint32 public immutable driverId;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L25

```solidity
File: src/ImmutableSplitsDriver.sol

15:       uint32 public immutable driverId;

17:       uint32 public immutable totalSplitsWeight;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L15

```solidity
File: src/NFTDriver.sol

25:       uint32 public immutable driverId;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L25

### [G&#x2011;07]  Use custom errors rather than `revert()`/`require()` strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

*There are 3 instances of this issue:*

```solidity
File: src/Managed.sol

53            require(
54                admin() == msg.sender || isPauser(msg.sender), "Caller is not the admin or a pauser"
55:           );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L53-L55

```solidity
File: src/NFTDriver.sol

41            require(
42                _isApprovedOrOwner(_msgSender(), tokenId),
43                "ERC721: caller is not token owner or approved"
44:           );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L41-L44

```solidity
File: src/Splits.sol

254           require(
255               _hashSplits(currReceivers) == _splitsHash(userId), "Invalid current splits receivers"
256:          );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L254-L256

### [G&#x2011;08]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 10 instances of this issue:*

```solidity
File: src/DripsHub.sol

386       function collect(uint256 userId, IERC20 erc20)
387           public
388           whenNotPaused
389           onlyDriver(userId)
390:          returns (uint128 amt)

409       function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)
410           public
411           whenNotPaused
412:          onlyDriver(userId)

510       function setDrips(
511           uint256 userId,
512           IERC20 erc20,
513           DripsReceiver[] memory currReceivers,
514           int128 balanceDelta,
515           DripsReceiver[] memory newReceivers,
516           // slither-disable-next-line similar-names
517           uint32 maxEndHint1,
518           uint32 maxEndHint2
519:      ) public whenNotPaused onlyDriver(userId) returns (int128 realBalanceDelta) {

576       function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
577           public
578           whenNotPaused
579:          onlyDriver(userId)

608       function emitUserMetadata(uint256 userId, UserMetadata[] calldata userMetadata)
609           public
610           whenNotPaused
611:          onlyDriver(userId)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L386-L390

```solidity
File: src/NFTDriver.sol

109       function collect(uint256 tokenId, IERC20 erc20, address transferTo)
110           public
111           whenNotPaused
112           onlyHolder(tokenId)
113:          returns (uint128 amt)

133       function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)
134           public
135           whenNotPaused
136:          onlyHolder(tokenId)

186       function setDrips(
187           uint256 tokenId,
188           IERC20 erc20,
189           DripsReceiver[] calldata currReceivers,
190           int128 balanceDelta,
191           DripsReceiver[] calldata newReceivers,
192           // slither-disable-next-line similar-names
193           uint32 maxEndHint1,
194           uint32 maxEndHint2,
195           address transferTo
196:      ) public whenNotPaused onlyHolder(tokenId) returns (int128 realBalanceDelta) {

220       function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)
221           public
222           whenNotPaused
223:          onlyHolder(tokenId)

235       function emitUserMetadata(uint256 tokenId, UserMetadata[] calldata userMetadata)
236           public
237           whenNotPaused
238:          onlyHolder(tokenId)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L109-L113

### [G&#x2011;09]  Constructors can be marked `payable`
Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. A constructor can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.

*There are 8 instances of this issue:*

```solidity
File: src/AddressDriver.sol

30        constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
31:           ERC2771Context(forwarder)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L30-L31

```solidity
File: src/DripsHub.sol

109       constructor(uint32 cycleSecs_)
110           Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
111:          Splits(_erc1967Slot("eip1967.splits.storage"))

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L109-L111

```solidity
File: src/Drips.sol

219:      constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L219

```solidity
File: src/ImmutableSplitsDriver.sol

28:       constructor(DripsHub _dripsHub, uint32 _driverId) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L28

```solidity
File: src/Managed.sol

73        constructor() {
74:           _managedStorage().isPaused = true;

158:      constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0)) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L73-L74

```solidity
File: src/NFTDriver.sol

32        constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
33            ERC2771Context(forwarder)
34:           ERC721("DripsHub identity", "DHI")

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L32-L34

```solidity
File: src/Splits.sol

94:       constructor(bytes32 splitsStorageSlot) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L94


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas | 13 |  1560 |
| [G&#x2011;02] | `<array>.length` should not be looked up in every loop of a `for`-loop | 11 |  33 |
| [G&#x2011;03] | `require()`/`revert()` strings longer than 32 bytes cost extra gas | 3 |  - |
| [G&#x2011;04] | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 21 |  105 |
| [G&#x2011;05] | Using `private` rather than `public` for constants, saves gas | 7 |  - |
| [G&#x2011;06] | Division by two should use bit shifting | 2 |  40 |
| [G&#x2011;07] | Use custom errors rather than `revert()`/`require()` strings to save gas | 27 |  - |
| [G&#x2011;08] | Functions guaranteed to revert when called by normal users can be marked `payable` | 6 |  126 |

Total: 90 instances over 8 issues with **1864 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. **Each iteration of this for-loop costs at least 60 gas** (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I've also flagged instances where the function is `public` but can be marked as `external` since it's not called by the contract, and cases where a constructor is involved

*There are 13 instances of this issue:*

```solidity
File: src/Caller.sol

/// @audit data - (valid but excluded finding)
144       function callAs(address sender, address to, bytes memory data)
145           public
146           payable
147:          returns (bytes memory returnData)

/// @audit data - (valid but excluded finding)
164       function callSigned(
165           address sender,
166           address to,
167           bytes memory data,
168           uint256 deadline,
169           bytes32 r,
170           bytes32 sv
171:      ) public payable returns (bytes memory returnData) {

/// @audit calls - (valid but excluded finding)
193:      function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L144-L147

```solidity
File: src/DripsHub.sol

/// @audit dripsHistory - (valid but excluded finding)
270       function squeezeDrips(
271           uint256 userId,
272           IERC20 erc20,
273           uint256 senderId,
274           bytes32 historyHash,
275           DripsHistory[] memory dripsHistory
276:      ) public whenNotPaused returns (uint128 amt) {

/// @audit dripsHistory - (valid but excluded finding)
297       function squeezeDripsResult(
298           uint256 userId,
299           IERC20 erc20,
300           uint256 senderId,
301           bytes32 historyHash,
302           DripsHistory[] memory dripsHistory
303:      ) public view returns (uint128 amt) {

/// @audit currReceivers - (valid but excluded finding)
328       function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
329           public
330           view
331:          returns (uint128 collectableAmt, uint128 splitAmt)

/// @audit currReceivers - (valid but excluded finding)
355       function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)
356           public
357           whenNotPaused
358:          returns (uint128 collectableAmt, uint128 splitAmt)

/// @audit receivers - (valid but excluded finding)
460       function balanceAt(
461           uint256 userId,
462           IERC20 erc20,
463           DripsReceiver[] memory receivers,
464           uint32 timestamp
465:      ) public view returns (uint128 balance) {

/// @audit currReceivers - (valid but excluded finding)
/// @audit newReceivers - (valid but excluded finding)
510       function setDrips(
511           uint256 userId,
512           IERC20 erc20,
513           DripsReceiver[] memory currReceivers,
514           int128 balanceDelta,
515           DripsReceiver[] memory newReceivers,
516           // slither-disable-next-line similar-names
517           uint32 maxEndHint1,
518           uint32 maxEndHint2
519:      ) public whenNotPaused onlyDriver(userId) returns (int128 realBalanceDelta) {

/// @audit receivers - (valid but excluded finding)
546:      function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {

/// @audit receivers - (valid but excluded finding)
576       function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
577           public
578           whenNotPaused
579:          onlyDriver(userId)

/// @audit receivers - (valid but excluded finding)
595       function hashSplits(SplitsReceiver[] memory receivers)
596           public
597           pure
598:          returns (bytes32 receiversHash)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L270-L276

### [G&#x2011;02]  `<array>.length` should not be looked up in every loop of a `for`-loop
The overheads outlined below are _PER LOOP_, excluding the first loop
* storage arrays incur a Gwarmaccess (**100 gas**)
* memory arrays use `MLOAD` (**3 gas**)
* calldata arrays use `CALLDATALOAD` (**3 gas**)

Caching the length changes each of these to a `DUP<N>` (**3 gas**), and gets rid of the extra `DUP<N>` needed to store the stack offset

*There are 11 instances of this issue:*

```solidity
File: src/Caller.sol

/// @audit (valid but excluded finding)
196:          for (uint256 i = 0; i < calls.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L196

```solidity
File: src/DripsHub.sol

/// @audit (valid but excluded finding)
613:          for (uint256 i = 0; i < userMetadata.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L613

```solidity
File: src/Drips.sol

/// @audit (valid but excluded finding)
450:          for (uint256 i = 0; i < dripsHistory.length; i++) {

/// @audit (valid but excluded finding)
490:          for (; idx < receivers.length; idx++) {

/// @audit (valid but excluded finding)
563:          for (uint256 i = 0; i < receivers.length; i++) {

/// @audit (valid but excluded finding)
664:              for (uint256 i = 0; i < newReceivers.length; i++) {

/// @audit (valid but excluded finding)
777:              for (uint256 i = 0; i < receivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L450

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit (valid but excluded finding)
61:           for (uint256 i = 0; i < receivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L61

```solidity
File: src/Splits.sol

/// @audit (valid but excluded finding)
127:          for (uint256 i = 0; i < currReceivers.length; i++) {

/// @audit (valid but excluded finding)
158:          for (uint256 i = 0; i < currReceivers.length; i++) {

/// @audit (valid but excluded finding)
231:          for (uint256 i = 0; i < receivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L127

### [G&#x2011;03]  `require()`/`revert()` strings longer than 32 bytes cost extra gas
Each extra memory word of bytes past the original 32 [incurs an MSTORE](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings) which costs **3 gas**

*There are 3 instances of this issue:*

```solidity
File: src/Drips.sol

/// @audit (valid but excluded finding)
454:                  require(dripsHash == 0, "Drips history entry with hash and receivers");

/// @audit (valid but excluded finding)
540:          require(timestamp >= state.updateTime, "Timestamp before last drips update");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L454

```solidity
File: src/Splits.sol

/// @audit (valid but excluded finding)
239:                  require(prevUserId < userId, "Splits receivers not sorted by user ID");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L239

### [G&#x2011;04]  `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
Saves **5 gas per loop**

*There are 21 instances of this issue:*

```solidity
File: src/Caller.sol

/// @audit (valid but excluded finding)
196:          for (uint256 i = 0; i < calls.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L196

```solidity
File: src/DripsHub.sol

/// @audit (valid but excluded finding)
613:          for (uint256 i = 0; i < userMetadata.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L613

```solidity
File: src/Drips.sol

/// @audit (valid but excluded finding)
247:              for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

/// @audit (valid but excluded finding)
287:          for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

/// @audit (valid but excluded finding)
358:          for (uint256 i = 0; i < squeezedNum; i++) {

/// @audit (valid but excluded finding)
422:          for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

/// @audit (valid but excluded finding)
450:          for (uint256 i = 0; i < dripsHistory.length; i++) {

/// @audit (valid but excluded finding)
490:          for (; idx < receivers.length; idx++) {

/// @audit (valid but excluded finding)
563:          for (uint256 i = 0; i < receivers.length; i++) {

/// @audit (valid but excluded finding)
655:              state.currCycleConfigs++;

/// @audit (valid but excluded finding)
664:              for (uint256 i = 0; i < newReceivers.length; i++) {

/// @audit (valid but excluded finding)
745:              for (uint256 i = 0; i < configsLen; i++) {

/// @audit (valid but excluded finding)
777:              for (uint256 i = 0; i < receivers.length; i++) {

/// @audit (valid but excluded finding)
959:                  currIdx++;

/// @audit (valid but excluded finding)
962:                  newIdx++;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L247

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit (valid but excluded finding)
59:           StorageSlot.getUint256Slot(_counterSlot).value++;

/// @audit (valid but excluded finding)
61:           for (uint256 i = 0; i < receivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L59

```solidity
File: src/NFTDriver.sol

/// @audit (valid but excluded finding)
93:           StorageSlot.getUint256Slot(_mintedTokensSlot).value++;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L93

```solidity
File: src/Splits.sol

/// @audit (valid but excluded finding)
127:          for (uint256 i = 0; i < currReceivers.length; i++) {

/// @audit (valid but excluded finding)
158:          for (uint256 i = 0; i < currReceivers.length; i++) {

/// @audit (valid but excluded finding)
231:          for (uint256 i = 0; i < receivers.length; i++) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L127

### [G&#x2011;05]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 7 instances of this issue:*

```solidity
File: src/DripsHub.sol

/// @audit (valid but excluded finding)
56:       uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;

/// @audit (valid but excluded finding)
58:       uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;

/// @audit (valid but excluded finding)
60:       uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;

/// @audit (valid but excluded finding)
62:       uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;

/// @audit (valid but excluded finding)
64:       uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;

/// @audit (valid but excluded finding)
67:       uint256 public constant DRIVER_ID_OFFSET = 224;

/// @audit (valid but excluded finding)
70:       uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L56

### [G&#x2011;06]  Division by two should use bit shifting
`<x> / 2` is the same as `<x> >> 1`. While the compiler uses the `SHR` opcode to accomplish both, the version that uses division incurs an overhead of [**20 gas**](https://gist.github.com/IllIllI000/ec0e4e6c4f52a6bca158f137a3afd4ff) due to `JUMP`s to and from a compiler utility function that introduces checks which can be avoided by using `unchecked {}` around the division by two

*There are 2 instances of this issue:*

```solidity
File: src/Drips.sol

/// @audit (valid but excluded finding)
480:              uint256 idxMid = (idx + idxCap) / 2;

/// @audit (valid but excluded finding)
717:                  uint256 end = (enoughEnd + notEnoughEnd) / 2;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L480

### [G&#x2011;07]  Use custom errors rather than `revert()`/`require()` strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

*There are 27 instances of this issue:*

```solidity
File: src/Caller.sol

/// @audit (valid but excluded finding)
108:          require(_authorized[sender].add(user), "Address already is authorized");

/// @audit (valid but excluded finding)
116:          require(_authorized[sender].remove(user), "Address is not authorized");

/// @audit (valid but excluded finding)
149:          require(isAuthorized(sender, _msgSender()), "Not authorized");

/// @audit (valid but excluded finding)
173:          require(block.timestamp <= deadline, "Execution deadline expired");

/// @audit (valid but excluded finding)
181:          require(signer == sender, "Invalid signature");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L108

```solidity
File: src/DripsHub.sol

/// @audit (valid but excluded finding)
125:          require(driverAddress(driverId) == msg.sender, "Callable only by the driver");

/// @audit (valid but excluded finding)
631:          require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L125

```solidity
File: src/Drips.sol

/// @audit (valid but excluded finding)
220:          require(cycleSecs > 1, "Cycle length too low");

/// @audit (valid but excluded finding)
454:                  require(dripsHash == 0, "Drips history entry with hash and receivers");

/// @audit (valid but excluded finding)
461:          require(historyHash == finalHistoryHash, "Invalid drips history");

/// @audit (valid but excluded finding)
540:          require(timestamp >= state.updateTime, "Timestamp before last drips update");

/// @audit (valid but excluded finding)
541:          require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list");

/// @audit (valid but excluded finding)
622:          require(_hashDrips(currReceivers) == state.dripsHash, "Invalid current drips list");

/// @audit (valid but excluded finding)
775:              require(receivers.length <= _MAX_DRIPS_RECEIVERS, "Too many drips receivers");

/// @audit (valid but excluded finding)
780:                      require(_isOrdered(receivers[i - 1], receiver), "Receivers not sorted");

/// @audit (valid but excluded finding)
798:          require(amtPerSec != 0, "Drips receiver amtPerSec is zero");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L220

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit (valid but excluded finding)
64:           require(weightSum == totalSplitsWeight, "Invalid total receivers weight");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L64

```solidity
File: src/Managed.sol

/// @audit (valid but excluded finding)
47:           require(admin() == msg.sender, "Caller is not the admin");

/// @audit (valid but excluded finding)
61:           require(!paused(), "Contract paused");

/// @audit (valid but excluded finding)
67:           require(paused(), "Contract not paused");

/// @audit (valid but excluded finding)
91:           require(_managedStorage().pausers.add(pauser), "Address already is a pauser");

/// @audit (valid but excluded finding)
98:           require(_managedStorage().pausers.remove(pauser), "Address is not a pauser");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L47

```solidity
File: src/Splits.sol

/// @audit (valid but excluded finding)
227:          require(receivers.length <= _MAX_SPLITS_RECEIVERS, "Too many splits receivers");

/// @audit (valid but excluded finding)
234:              require(weight != 0, "Splits receiver weight is zero");

/// @audit (valid but excluded finding)
238:                  require(prevUserId != userId, "Duplicate splits receivers");

/// @audit (valid but excluded finding)
239:                  require(prevUserId < userId, "Splits receivers not sorted by user ID");

/// @audit (valid but excluded finding)
244:          require(totalWeight <= _TOTAL_SPLITS_WEIGHT, "Splits weights sum too high");

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L227

### [G&#x2011;08]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 6 instances of this issue:*

```solidity
File: src/Managed.sol

/// @audit (valid but excluded finding)
84:       function changeAdmin(address newAdmin) public onlyAdmin {

/// @audit (valid but excluded finding)
90:       function grantPauser(address pauser) public onlyAdmin {

/// @audit (valid but excluded finding)
97:       function revokePauser(address pauser) public onlyAdmin {

/// @audit (valid but excluded finding)
122:      function pause() public onlyAdminOrPauser whenNotPaused {

/// @audit (valid but excluded finding)
128:      function unpause() public onlyAdminOrPauser whenPaused {

/// @audit (valid but excluded finding)
151:      function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L84
