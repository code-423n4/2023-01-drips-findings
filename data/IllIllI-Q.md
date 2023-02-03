## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Value may overflow bits into `driverId` portion | 1 | 
| [L&#x2011;02] | Use of `msg.value` in a loop | 1 | 
| [L&#x2011;03] | Not all tokens support approve-max | 1 | 
| [L&#x2011;04] | Non-upgradeable version of contract in use | 1 | 
| [L&#x2011;05] | No trusted forwarder setup | 1 | 
| [L&#x2011;06] | Draft imports may break in new minor versions | 1 | 

Total: 6 instances over 6 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | `2**<n> - 1` should be re-written as `type(uint<n>).max` | 3 | 
| [N&#x2011;02] | `constant`s should be defined rather than using magic numbers | 18 | 
| [N&#x2011;03] | Events that mark critical parameter changes should contain both the old and the new value | 2 | 
| [N&#x2011;04] | Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant` | 1 | 
| [N&#x2011;05] | Constant redefined elsewhere | 4 | 
| [N&#x2011;06] | Non-library/interface files should use fixed compiler versions, not floating ones | 6 | 
| [N&#x2011;07] | NatSpec is incomplete | 7 | 
| [N&#x2011;08] | Not using the named return variables anywhere in the function is confusing | 58 | 
| [N&#x2011;09] | Consider using `delete` rather than assigning zero to clear values | 2 | 
| [N&#x2011;10] | Large assembly blocks should have extensive comments | 1 | 
| [N&#x2011;11] | Contracts should have full test coverage | 1 | 
| [N&#x2011;12] | Large or complicated code bases should implement fuzzing tests | 1 | 
| [N&#x2011;13] | Function ordering does not follow the Solidity style guide | 10 | 
| [N&#x2011;14] | Contract does not follow the Solidity style guide's suggested layout ordering | 3 | 

Total: 117 instances over 14 issues





## Low Risk Issues

### [L&#x2011;01]  Value may overflow bits into `driverId` portion
If a lot of NFTs are minted, it's possible for the next value to be larger than the expected number of bits, and when it's added to the `driverId`, the driverId will be corrupted such that it no longer will match the correct driver, and any funds sent to that address will be stuck. Should ensure that the value never gets that high with a `require()`

*There is 1 instance of this issue:*

```solidity
File: /src/NFTDriver.sol

51:          return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_mintedTokensSlot).value;

```
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L51

### [L&#x2011;02]  Use of `msg.value` in a loop
The `callBatched()` function does not keep track of how much of msg.value has been spent for each iteration. If there are any latent funds, or if the contract has an upgrade where it stores an Ether balance, the funds will be taken.

*There is 1 instance of this issue:*

```solidity
File: /src/Caller.sol

193:     function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {

```
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L193

### [L&#x2011;03]  Not all tokens support approve-max
Some tokens consider uint256 as just another value. If one of these tokens is used and the approval balance eventually reaches only a couple wei, nobody will be able to send funds because there won't be enough approval

*There is 1 instance of this issue:*

```solidity
File: /src/AddressDriver.sol

174:             erc20.safeApprove(address(dripsHub), type(uint256).max);

```
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L174

### [L&#x2011;04]  Non-upgradeable version of contract in use
Use [ERC721BurnableUpgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC721/extensions/ERC721BurnableUpgradeable.sol) rather than ERC721Burnable

*There is 1 instance of this issue:*

```solidity
File: /src/NFTDriver.sol

19:  contract NFTDriver is ERC721Burnable, ERC2771Context, Managed {

```
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L19

### [L&#x2011;05]  No trusted forwarder setup
Use of ERC2771 requires the use of a `trustedForwarder()` function, and without one, use of `_msgSender()` is a waste of gas. If you intend to support the feature, have a default implementation that returns false, and direct extenders to override it

*There is 1 instance of this issue:*

```solidity
File: /src/AddressDriver.sol

19:  contract AddressDriver is Managed, ERC2771Context {

```
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L19

### [L&#x2011;06]  Draft imports may break in new minor versions
While OpenZeppelin draft contracts are safe to use and have been audited, their 'draft' status means that the EIPs they're based on are not finalized, and thus there may be breaking changes in even [minor releases](https://docs.openzeppelin.com/contracts/3.x/api/drafts). If a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to has breaking changes in the new version, you'll encounter unnecessary delays in porting and testing replacement contracts. Ensure that you have extensive test coverage of this area so that differences can be automatically detected, and have a plan in place for how you would fully test a new version of these contracts if they do indeed change unexpectedly. Consider creating a forked version of the file rather than importing it from the package, and manually patch your fork as changes are made.

*There is 1 instance of this issue:*

```solidity
File: src/Caller.sol

5:    import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L5

## Non-critical Issues

### [N&#x2011;01]  `2**<n> - 1` should be re-written as `type(uint<n>).max`
Earlier versions of solidity can use `uint<n>(-1)` instead. Expressions not including the `- 1` can often be re-written to accomodate the change (e.g. by using a `>` rather than a `>=`, which will also save some gas)

*There are 3 instances of this issue:*

```solidity
File: src/Drips.sol

185:          mapping(uint256 => uint32[2 ** 32]) nextSqueezed;

357:          uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];

419:          uint32[2 ** 32] storage nextSqueezed =

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L185

### [N&#x2011;02]  `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*There are 18 instances of this issue:*

```solidity
File: src/AddressDriver.sol

/// @audit 224
41:           return (uint256(driverId) << 224) | uint160(userAddr);

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L41

```solidity
File: src/Drips.sol

/// @audit 160
66:           config = (config << 160) | amtPerSec_;

/// @audit 32
67:           config = (config << 32) | start_;

/// @audit 32
68:           config = (config << 32) | duration_;

/// @audit 224
74:           return uint32(DripsConfig.unwrap(config) >> 224);

/// @audit 64
79:           return uint160(DripsConfig.unwrap(config) >> 64);

/// @audit 32
84:           return uint32(DripsConfig.unwrap(config) >> 32);

/// @audit 32
185:          mapping(uint256 => uint32[2 ** 32]) nextSqueezed;

/// @audit 32
357:          uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];

/// @audit 32
419:          uint32[2 ** 32] storage nextSqueezed =

/// @audit 64
/// @audit 32
805:          configs[configsLen] = (amtPerSec << 64) | (start << 32) | end;

/// @audit 32
/// @audit 5
823:              val := mload(add(32, add(configs, shl(5, idx))))

/// @audit 64
/// @audit 32
825:          return (val >> 64, uint32(val >> 32), uint32(val));

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L66

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit 224
37:           return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_counterSlot).value;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L37

```solidity
File: src/NFTDriver.sol

/// @audit 224
51:           return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_mintedTokensSlot).value;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L51

### [N&#x2011;03]  Events that mark critical parameter changes should contain both the old and the new value
This should especially be done if the new value is not required to be different from the old value

*There are 2 instances of this issue:*

```solidity
File: src/DripsHub.sol

/// @audit registerDriver()
138:          emit DriverRegistered(driverId, driverAddr);

/// @audit updateDriverAddress()
155:          emit DriverAddressUpdated(driverId, msg.sender, newDriverAddr);

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L138

### [N&#x2011;04]  Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant`
While it **doesn't save any gas** because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constants` should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the constructor.

*There is 1 instance of this issue:*

```solidity
File: src/Caller.sol

84:       bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L84

### [N&#x2011;05]  Constant redefined elsewhere
Consider defining in only one contract so that values cannot become out of sync when only one location is updated. A [cheap way](https://medium.com/coinmonks/gas-cost-of-solidity-library-functions-dbe0cedd4678) to store constants in a single location is to create an `internal constant` in a `library`. If the variable is a local cache of another contract's value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don't get out of sync.

*There are 4 instances of this issue:*

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit seen in src/AddressDriver.sol 
13:       DripsHub public immutable dripsHub;

/// @audit seen in src/AddressDriver.sol 
15:       uint32 public immutable driverId;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L13

```solidity
File: src/NFTDriver.sol

/// @audit seen in src/ImmutableSplitsDriver.sol 
23:       DripsHub public immutable dripsHub;

/// @audit seen in src/ImmutableSplitsDriver.sol 
25:       uint32 public immutable driverId;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L23

### [N&#x2011;06]  Non-library/interface files should use fixed compiler versions, not floating ones

*There are 6 instances of this issue:*

```solidity
File: src/AddressDriver.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L2

```solidity
File: src/Caller.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L2

```solidity
File: src/DripsHub.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L2

```solidity
File: src/ImmutableSplitsDriver.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L2

```solidity
File: src/Managed.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L2

```solidity
File: src/NFTDriver.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L2

### [N&#x2011;07]  NatSpec is incomplete

*There are 7 instances of this issue:*

```solidity
File: src/Drips.sol

/// @audit Missing: '@return'
58        /// @param duration_ The duration of dripping.
59        /// If zero, drip until balance runs out.
60        function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)
61            internal
62            pure
63:           returns (DripsConfig)

/// @audit Missing: '@param updateTime'
/// @audit Missing: '@return'
967       /// @notice Calculates the time range in the future in which a receiver will be dripped to.
968       /// @param receiver The drips receiver
969       /// @param maxEnd The maximum end time of drips
970       function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)
971           private
972           view
973:          returns (uint32 start, uint32 end)

/// @audit Missing: '@return'
983       /// @param startCap The timestamp the drips range start should be capped to
984       /// @param endCap The timestamp the drips range end should be capped to
985       function _dripsRange(
986           DripsReceiver memory receiver,
987           uint32 updateTime,
988           uint32 maxEnd,
989           uint32 startCap,
990           uint32 endCap
991:      ) private pure returns (uint32 start, uint32 end_) {

/// @audit Missing: '@param next'
/// @audit Missing: '@return'
1058      /// @notice Checks if two receivers fulfil the sortedness requirement of the receivers list.
1059      /// @param prev The previous receiver
1060      /// @param prev The next receiver
1061      function _isOrdered(DripsReceiver memory prev, DripsReceiver memory next)
1062          private
1063          pure
1064:         returns (bool)

/// @audit Missing: '@return'
1091      /// @param end The dripping end time
1092      /// @param amt The dripped amount
1093      function _drippedAmt(uint256 amtPerSec, uint256 start, uint256 end)
1094          private
1095          view
1096:         returns (uint256 amt)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L58-L63

### [N&#x2011;08]  Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one

*There are 58 instances of this issue:*

```solidity
File: src/AddressDriver.sol

/// @audit userId
40:       function calcUserId(address userAddr) public view returns (uint256 userId) {

/// @audit userId
46:       function callerUserId() internal view returns (uint256 userId) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L40

```solidity
File: src/Caller.sol

/// @audit authorized
124:      function isAuthorized(address sender, address user) public view returns (bool authorized) {

/// @audit authorized
132:      function allAuthorized(address sender) public view returns (address[] memory authorized) {

/// @audit returnData
144       function callAs(address sender, address to, bytes memory data)
145           public
146           payable
147:          returns (bytes memory returnData)

/// @audit returnData
164       function callSigned(
165           address sender,
166           address to,
167           bytes memory data,
168           uint256 deadline,
169           bytes32 r,
170           bytes32 sv
171:      ) public payable returns (bytes memory returnData) {

/// @audit returnData
202       function _call(address sender, address to, bytes memory data, uint256 value)
203           internal
204:          returns (bytes memory returnData)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L124

```solidity
File: src/DripsHub.sol

/// @audit driverAddr
145:      function driverAddress(uint32 driverId) public view returns (address driverAddr) {

/// @audit driverId
160:      function nextDriverId() public view returns (uint32 driverId) {

/// @audit cycleSecs_
169:      function cycleSecs() public view returns (uint32 cycleSecs_) {

/// @audit balance
181:      function totalBalance(IERC20 erc20) public view returns (uint256 balance) {

/// @audit cycles
196       function receivableDripsCycles(uint256 userId, IERC20 erc20)
197           public
198           view
199:          returns (uint32 cycles)

/// @audit amt
317:      function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

/// @audit collectableAmt
/// @audit splitAmt
328       function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
329           public
330           view
331:          returns (uint128 collectableAmt, uint128 splitAmt)

/// @audit collectableAmt
/// @audit splitAmt
355       function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)
356           public
357           whenNotPaused
358:          returns (uint128 collectableAmt, uint128 splitAmt)

/// @audit amt
372:      function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

/// @audit dripsHash
/// @audit dripsHistoryHash
/// @audit updateTime
/// @audit balance
/// @audit maxEnd
432       function dripsState(uint256 userId, IERC20 erc20)
433           public
434           view
435           returns (
436               bytes32 dripsHash,
437               bytes32 dripsHistoryHash,
438               uint32 updateTime,
439               uint128 balance,
440:              uint32 maxEnd

/// @audit balance
460       function balanceAt(
461           uint256 userId,
462           IERC20 erc20,
463           DripsReceiver[] memory receivers,
464           uint32 timestamp
465:      ) public view returns (uint128 balance) {

/// @audit dripsHash
546:      function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {

/// @audit dripsHistoryHash
557       function hashDripsHistory(
558           bytes32 oldDripsHistoryHash,
559           bytes32 dripsHash,
560           uint32 updateTime,
561           uint32 maxEnd
562:      ) public pure returns (bytes32 dripsHistoryHash) {

/// @audit currSplitsHash
587:      function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {

/// @audit receiversHash
595       function hashSplits(SplitsReceiver[] memory receivers)
596           public
597           pure
598:          returns (bytes32 receiversHash)

/// @audit assetId
642:      function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L145

```solidity
File: src/Drips.sol

/// @audit cycles
300       function _receivableDripsCycles(uint256 userId, uint256 assetId)
301           internal
302           view
303:          returns (uint32 cycles)

/// @audit squeezedAmt
470       function _squeezedAmt(
471           uint256 userId,
472           DripsHistory memory dripsHistory,
473           uint32 squeezeStartCap,
474           uint32 squeezeEndCap
475:      ) private view returns (uint128 squeezedAmt) {

/// @audit dripsHash
/// @audit dripsHistoryHash
/// @audit updateTime
/// @audit balance
/// @audit maxEnd
508       function _dripsState(uint256 userId, uint256 assetId)
509           internal
510           view
511           returns (
512               bytes32 dripsHash,
513               bytes32 dripsHistoryHash,
514               uint32 updateTime,
515               uint128 balance,
516:              uint32 maxEnd

/// @audit balance
533       function _balanceAt(
534           uint256 userId,
535           uint256 assetId,
536           DripsReceiver[] memory receivers,
537           uint32 timestamp
538:      ) internal view returns (uint128 balance) {

/// @audit newConfigsLen
792       function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver)
793           private
794           view
795:          returns (uint256 newConfigsLen)

/// @audit amtPerSec
/// @audit start
/// @audit end
815       function _getConfig(uint256[] memory configs, uint256 idx)
816           private
817           pure
818:          returns (uint256 amtPerSec, uint256 start, uint256 end)

/// @audit dripsHash
834       function _hashDrips(DripsReceiver[] memory receivers)
835           internal
836           pure
837:          returns (bytes32 dripsHash)

/// @audit dripsHistoryHash
852       function _hashDripsHistory(
853           bytes32 oldDripsHistoryHash,
854           bytes32 dripsHash,
855           uint32 updateTime,
856           uint32 maxEnd
857:      ) internal pure returns (bytes32 dripsHistoryHash) {

/// @audit start
/// @audit end
970       function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)
971           private
972           view
973:          returns (uint32 start, uint32 end)

/// @audit end_
985       function _dripsRange(
986           DripsReceiver memory receiver,
987           uint32 updateTime,
988           uint32 maxEnd,
989           uint32 startCap,
990           uint32 endCap
991:      ) private pure returns (uint32 start, uint32 end_) {

/// @audit timestamp
1128:     function _currTimestamp() private view returns (uint32 timestamp) {

/// @audit timestamp
1134:     function _currCycleStart() private view returns (uint32 timestamp) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L300-L303

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit userId
36:       function nextUserId() public view returns (uint256 userId) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L36

```solidity
File: src/Managed.sol

/// @audit isAddrPauser
105:      function isPauser(address pauser) public view returns (bool isAddrPauser) {

/// @audit pausersList
112:      function allPausers() public view returns (address[] memory pausersList) {

/// @audit isPaused
117:      function paused() public view returns (bool isPaused) {

/// @audit slot
136:      function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L105

```solidity
File: src/NFTDriver.sol

/// @audit tokenId
50:       function nextTokenId() public view returns (uint256 tokenId) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L50

```solidity
File: src/Splits.sol

/// @audit amt
106:      function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {

/// @audit amt
176:      function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {

/// @audit currSplitsHash
262:      function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {

/// @audit receiversHash
270       function _hashSplits(SplitsReceiver[] memory receivers)
271           internal
272           pure
273:          returns (bytes32 receiversHash)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L106

### [N&#x2011;09]  Consider using `delete` rather than assigning zero to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic

*There are 2 instances of this issue:*

```solidity
File: src/Splits.sol

156:          balance.splittable = 0;

188:          balance.collectable = 0;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L156

### [N&#x2011;10]  Large assembly blocks should have extensive comments
Assembly blocks are take a lot more time to audit than normal Solidity code, and often have gotchas and side-effects that the Solidity versions of the same code do not. Consider adding more comments explaining what is being done in every step of the assembly code

*There is 1 instance of this issue:*

```solidity
File: src/Drips.sol

1103          assembly {
1104              let endedCycles := sub(div(end, cycleSecs), div(start, cycleSecs))
1105              // slither-disable-next-line divide-before-multiply
1106              let amtPerCycle := div(mul(cycleSecs, amtPerSec), _AMT_PER_SEC_MULTIPLIER)
1107              amt := mul(endedCycles, amtPerCycle)
1108              // slither-disable-next-line weak-prng
1109              let amtEnd := div(mul(mod(end, cycleSecs), amtPerSec), _AMT_PER_SEC_MULTIPLIER)
1110              amt := add(amt, amtEnd)
1111              // slither-disable-next-line weak-prng
1112              let amtStart := div(mul(mod(start, cycleSecs), amtPerSec), _AMT_PER_SEC_MULTIPLIER)
1113              amt := sub(amt, amtStart)
1114:         }

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L1103-L1114

### [N&#x2011;11]  Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```

### [N&#x2011;12]  Large or complicated code bases should implement fuzzing tests
Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement [fuzzing tests](https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05). Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```

### [N&#x2011;13]  Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

*There are 10 instances of this issue:*

```solidity
File: src/AddressDriver.sol

/// @audit callerUserId() came earlier
60:       function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L60

```solidity
File: src/DripsHub.sol

/// @audit _assertCallerIsDriver() came earlier
134:      function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L134

```solidity
File: src/Drips.sol

/// @audit lt() came earlier
219:      constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {

/// @audit _receivableDripsCyclesRange() came earlier
342       function _squeezeDrips(
343           uint256 userId,
344           uint256 assetId,
345           uint256 senderId,
346           bytes32 historyHash,
347           DripsHistory[] memory dripsHistory
348:      ) internal returns (uint128 amt) {

/// @audit _squeezedAmt() came earlier
508       function _dripsState(uint256 userId, uint256 assetId)
509           internal
510           view
511           returns (
512               bytes32 dripsHash,
513               bytes32 dripsHistoryHash,
514               uint32 updateTime,
515               uint128 balance,
516:              uint32 maxEnd

/// @audit _balanceAt() came earlier
610       function _setDrips(
611           uint256 userId,
612           uint256 assetId,
613           DripsReceiver[] memory currReceivers,
614           int128 balanceDelta,
615           DripsReceiver[] memory newReceivers,
616           // slither-disable-next-line similar-names
617           uint32 maxEndHint1,
618           uint32 maxEndHint2
619:      ) internal returns (int128 realBalanceDelta) {

/// @audit _getConfig() came earlier
834       function _hashDrips(DripsReceiver[] memory receivers)
835           internal
836           pure
837:          returns (bytes32 dripsHash)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L219

```solidity
File: src/Managed.sol

/// @audit _authorizeUpgrade() came earlier
158:      constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0)) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L158

```solidity
File: src/NFTDriver.sol

/// @audit _registerTokenId() came earlier
109       function collect(uint256 tokenId, IERC20 erc20, address transferTo)
110           public
111           whenNotPaused
112           onlyHolder(tokenId)
113:          returns (uint128 amt)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L109-L113

```solidity
File: src/Splits.sol

/// @audit _assertSplitsValid() came earlier
250:      function _assertCurrSplits(uint256 userId, SplitsReceiver[] memory currReceivers)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L250

### [N&#x2011;14]  Contract does not follow the Solidity style guide's suggested layout ordering
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be 1) Type declarations, 2) State variables, 3) Events, 4) Modifiers, and 5) Functions, but the contract(s) below do not follow this ordering

*There are 3 instances of this issue:*

```solidity
File: src/DripsHub.sol

/// @audit function constructor came earlier
118       modifier onlyDriver(uint256 userId) {
119           uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
120           _assertCallerIsDriver(driverId);
121           _;
122:      }

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L118-L122

```solidity
File: src/Drips.sol

/// @audit function lt came earlier
106:      uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L106

```solidity
File: src/NFTDriver.sol

/// @audit function constructor came earlier
40        modifier onlyHolder(uint256 tokenId) {
41            require(
42                _isApprovedOrOwner(_msgSender(), tokenId),
43                "ERC721: caller is not token owner or approved"
44            );
45            _;
46:       }

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L40-L46


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | `safeApprove()` is deprecated | 2 | 

Total: 2 instances over 1 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | `public` functions not called by the contract should be declared `external` instead | 44 | 
| [N&#x2011;02] | Event is missing `indexed` fields | 10 | 

Total: 54 instances over 2 issues





## Low Risk Issues

### [L&#x2011;01]  `safeApprove()` is deprecated
[Deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45) in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead. The function may currently work, but if a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to no longer has this function, you'll encounter unnecessary delays in porting and testing replacement contracts.

*There are 2 instances of this issue:*

```solidity
File: src/AddressDriver.sol

/// @audit (valid but excluded finding)
174:              erc20.safeApprove(address(dripsHub), type(uint256).max);

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L174

```solidity
File: src/NFTDriver.sol

/// @audit (valid but excluded finding)
289:              erc20.safeApprove(address(dripsHub), type(uint256).max);

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L289

## Non-critical Issues

### [N&#x2011;01]  `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There are 44 instances of this issue:*

```solidity
File: src/AddressDriver.sol

/// @audit (valid but excluded finding)
60:       function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {

/// @audit (valid but excluded finding)
76:       function give(uint256 receiver, IERC20 erc20, uint128 amt) public whenNotPaused {

/// @audit (valid but excluded finding)
122       function setDrips(
123           IERC20 erc20,
124           DripsReceiver[] calldata currReceivers,
125           int128 balanceDelta,
126           DripsReceiver[] calldata newReceivers,
127           // slither-disable-next-line similar-names
128           uint32 maxEndHint1,
129           uint32 maxEndHint2,
130           address transferTo
131:      ) public whenNotPaused returns (int128 realBalanceDelta) {

/// @audit (valid but excluded finding)
158:      function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {

/// @audit (valid but excluded finding)
166:      function emitUserMetadata(UserMetadata[] calldata userMetadata) public whenNotPaused {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/AddressDriver.sol#L60

```solidity
File: src/Caller.sol

/// @audit (valid but excluded finding)
106:      function authorize(address user) public {

/// @audit (valid but excluded finding)
114:      function unauthorize(address user) public {

/// @audit (valid but excluded finding)
132:      function allAuthorized(address sender) public view returns (address[] memory authorized) {

/// @audit (valid but excluded finding)
144       function callAs(address sender, address to, bytes memory data)
145           public
146           payable
147:          returns (bytes memory returnData)

/// @audit (valid but excluded finding)
164       function callSigned(
165           address sender,
166           address to,
167           bytes memory data,
168           uint256 deadline,
169           bytes32 r,
170           bytes32 sv
171:      ) public payable returns (bytes memory returnData) {

/// @audit (valid but excluded finding)
193:      function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol#L106

```solidity
File: src/DripsHub.sol

/// @audit (valid but excluded finding)
134:      function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {

/// @audit (valid but excluded finding)
152:      function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {

/// @audit (valid but excluded finding)
160:      function nextDriverId() public view returns (uint32 driverId) {

/// @audit (valid but excluded finding)
169:      function cycleSecs() public view returns (uint32 cycleSecs_) {

/// @audit (valid but excluded finding)
181:      function totalBalance(IERC20 erc20) public view returns (uint256 balance) {

/// @audit (valid but excluded finding)
196       function receivableDripsCycles(uint256 userId, IERC20 erc20)
197           public
198           view
199:          returns (uint32 cycles)

/// @audit (valid but excluded finding)
216       function receiveDripsResult(uint256 userId, IERC20 erc20, uint32 maxCycles)
217           public
218           view
219:          returns (uint128 receivableAmt)

/// @audit (valid but excluded finding)
238       function receiveDrips(uint256 userId, IERC20 erc20, uint32 maxCycles)
239           public
240           whenNotPaused
241:          returns (uint128 receivedAmt)

/// @audit (valid but excluded finding)
270       function squeezeDrips(
271           uint256 userId,
272           IERC20 erc20,
273           uint256 senderId,
274           bytes32 historyHash,
275           DripsHistory[] memory dripsHistory
276:      ) public whenNotPaused returns (uint128 amt) {

/// @audit (valid but excluded finding)
297       function squeezeDripsResult(
298           uint256 userId,
299           IERC20 erc20,
300           uint256 senderId,
301           bytes32 historyHash,
302           DripsHistory[] memory dripsHistory
303:      ) public view returns (uint128 amt) {

/// @audit (valid but excluded finding)
317:      function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

/// @audit (valid but excluded finding)
328       function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
329           public
330           view
331:          returns (uint128 collectableAmt, uint128 splitAmt)

/// @audit (valid but excluded finding)
355       function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)
356           public
357           whenNotPaused
358:          returns (uint128 collectableAmt, uint128 splitAmt)

/// @audit (valid but excluded finding)
372:      function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {

/// @audit (valid but excluded finding)
386       function collect(uint256 userId, IERC20 erc20)
387           public
388           whenNotPaused
389           onlyDriver(userId)
390:          returns (uint128 amt)

/// @audit (valid but excluded finding)
409       function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)
410           public
411           whenNotPaused
412:          onlyDriver(userId)

/// @audit (valid but excluded finding)
432       function dripsState(uint256 userId, IERC20 erc20)
433           public
434           view
435           returns (
436               bytes32 dripsHash,
437               bytes32 dripsHistoryHash,
438               uint32 updateTime,
439               uint128 balance,
440:              uint32 maxEnd

/// @audit (valid but excluded finding)
460       function balanceAt(
461           uint256 userId,
462           IERC20 erc20,
463           DripsReceiver[] memory receivers,
464           uint32 timestamp
465:      ) public view returns (uint128 balance) {

/// @audit (valid but excluded finding)
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

/// @audit (valid but excluded finding)
546:      function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {

/// @audit (valid but excluded finding)
557       function hashDripsHistory(
558           bytes32 oldDripsHistoryHash,
559           bytes32 dripsHash,
560           uint32 updateTime,
561           uint32 maxEnd
562:      ) public pure returns (bytes32 dripsHistoryHash) {

/// @audit (valid but excluded finding)
576       function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
577           public
578           whenNotPaused
579:          onlyDriver(userId)

/// @audit (valid but excluded finding)
587:      function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {

/// @audit (valid but excluded finding)
595       function hashSplits(SplitsReceiver[] memory receivers)
596           public
597           pure
598:          returns (bytes32 receiversHash)

/// @audit (valid but excluded finding)
608       function emitUserMetadata(uint256 userId, UserMetadata[] calldata userMetadata)
609           public
610           whenNotPaused
611:          onlyDriver(userId)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L134

```solidity
File: src/ImmutableSplitsDriver.sol

/// @audit (valid but excluded finding)
53        function createSplits(SplitsReceiver[] calldata receivers, UserMetadata[] calldata userMetadata)
54            public
55            whenNotPaused
56:           returns (uint256 userId)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/ImmutableSplitsDriver.sol#L53-L56

```solidity
File: src/NFTDriver.sol

/// @audit (valid but excluded finding)
62        function mint(address to, UserMetadata[] calldata userMetadata)
63            public
64            whenNotPaused
65:           returns (uint256 tokenId)

/// @audit (valid but excluded finding)
79        function safeMint(address to, UserMetadata[] calldata userMetadata)
80            public
81            whenNotPaused
82:           returns (uint256 tokenId)

/// @audit (valid but excluded finding)
109       function collect(uint256 tokenId, IERC20 erc20, address transferTo)
110           public
111           whenNotPaused
112           onlyHolder(tokenId)
113:          returns (uint128 amt)

/// @audit (valid but excluded finding)
133       function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)
134           public
135           whenNotPaused
136:          onlyHolder(tokenId)

/// @audit (valid but excluded finding)
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

/// @audit (valid but excluded finding)
220       function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)
221           public
222           whenNotPaused
223:          onlyHolder(tokenId)

/// @audit (valid but excluded finding)
235       function emitUserMetadata(uint256 tokenId, UserMetadata[] calldata userMetadata)
236           public
237           whenNotPaused
238:          onlyHolder(tokenId)

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/NFTDriver.sol#L62-L65

### [N&#x2011;02]  Event is missing `indexed` fields
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*There are 10 instances of this issue:*

```solidity
File: src/DripsHub.sol

/// @audit (valid but excluded finding)
93:       event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/DripsHub.sol#L93

```solidity
File: src/Drips.sol

/// @audit (valid but excluded finding)
143       event DripsReceiverSeen(
144           bytes32 indexed receiversHash, uint256 indexed userId, DripsConfig config
145:      );

/// @audit (valid but excluded finding)
152       event ReceivedDrips(
153           uint256 indexed userId, uint256 indexed assetId, uint128 amt, uint32 receivableCycles
154:      );

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Drips.sol#L143-L145

```solidity
File: src/Managed.sol

/// @audit (valid but excluded finding)
25:       event PauserGranted(address indexed pauser, address admin);

/// @audit (valid but excluded finding)
30:       event PauserRevoked(address indexed pauser, address admin);

/// @audit (valid but excluded finding)
34:       event Paused(address pauser);

/// @audit (valid but excluded finding)
38:       event Unpaused(address pauser);

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol#L25

```solidity
File: src/Splits.sol

/// @audit (valid but excluded finding)
33:       event Collected(uint256 indexed userId, uint256 indexed assetId, uint128 collected);

/// @audit (valid but excluded finding)
49:       event Collectable(uint256 indexed userId, uint256 indexed assetId, uint128 amt);

/// @audit (valid but excluded finding)
71:       event SplitsReceiverSeen(bytes32 indexed receiversHash, uint256 indexed userId, uint32 weight);

```
https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol#L33
