### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | Starting from Solidity '0.8.13', typing 'unchecked' increases gas cost by 3x |10 |
| [G-02] |Use hardcode address instead ``address(this)`` |7 |
| [G-03] |Using constants without caching saves gas |1 |
| [G-04] |Setting a loop interval instead of a "while" loop causes gas saved|1 |
| [G-05] |Gas overflow during iteration (DoS) |1|
| [G-06] |Using both `mint` and `safeMint` method at the same time is gas wasted method |1|
| [G-07] |Using storage instead of memory for structs/arrays saves gas |5|
| [G-08] |Remove unused codes |1|
| [G-09] |Using ``delete`` instead of setting  struct to 0 saves gas |2|
| [G-10] |Not using the named return variables when a function returns, wastes deployment gas|1|
| [G-11] |Don't use _msgSender() if not supporting EIP-2771|10|
| [G-12] |Use nested if and, avoid multiple check combinations |5|
| [G-13] |Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead |29|
| [G-14] |Sort Solidity operations using short-circuit mode |5|
| [G-15] |Setting the _constructor_ to `payable` |8|
| [G-16] |Optimize names to save gas |All Contracts|
| [G-17] |Use Minimalist and gas efficient standard ``ERC721`` implementation ||

Total 17 issues

### [G-01] Starting from Solidity '0.8.13', typing 'unchecked' increases gas cost by 3x

10 results - 5 files:
```solidity
src\Drips.sol:

  1103:    assembly {
  1104         let endedCycles := sub(div(end, cycleSecs), div(start, cycleSecs))

  1145:    assembly {
  1146         dripsStorage.slot := slot
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1103

```solidity
src\DripsHub.sol:

  624:     assembly {
  625          storageRef.slot := slot
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L624

```solidity
src\Managed.sol:

  145:     assembly {
  146          storageRef.slot := slot
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L145

```solidity
src\Splits.sol:

  286:     assembly {
  287          splitsStorage.slot := slot
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L286

```solidity
src\Drips.sol:

   686:    unchecked {
   687         (uint256[] memory configs, uint256 configsLen) = _buildConfigs(receivers);

   743:    unchecked {
   744         uint256 spent = 0;

   774:    unchecked {
   775         require(receivers.length <= _MAX_DRIPS_RECEIVERS, "Too many drips receivers");

  1041:    unchecked {

  1121:    unchecked {
  1122         return timestamp / _cycleSecs + 1;
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L686


### [G-02] Use hardcode address instead ``address(this)``

Instead of ``address(this)``, it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's ``script.sol`` and solmate ``LibRlp.sol`` contracts can do this.

Reference:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824


7 results - 4 files:
```solidity
src\AddressDriver.sol:

  171:    erc20.safeTransferFrom(_msgSender(), address(this), amt);

  173:    if (erc20.allowance(address(this), address(dripsHub)) == 0) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L171


```solidity
src\Caller.sol:

  81:     contract Caller is EIP712("Caller", "1"), ERC2771Context(address(this)) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L81

```solidity
src\DripsHub.sol:

  416:    erc20.safeTransferFrom(msg.sender, address(this), amt);

  533:    erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L416


```solidity
src\NFTDriver.sol:

  286:    erc20.safeTransferFrom(_msgSender(), address(this), amt);

  288:    if (erc20.allowance(address(this), address(dripsHub)) == 0) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L286

### [G-03] Using constants without caching saves gas

As seen in the code below, using constants without caching saves around 8k gas in deployment cost.

```diff
src\Drips.sol:
  1036:     function _addDelta(
  1037:         mapping(uint32 => AmtDelta) storage amtDeltas,
  1038:         uint256 timestamp,
  1039:         int256 amtPerSec
  1040:     ) private {
  1041:         unchecked {
  1042:             // In order to set a delta on a specific timestamp it must be introduced in two cycles.
  1043:             // These formulas follow the logic from `_drippedAmt`, see it for more details.
- 1044:             int256 amtPerSecMultiplier = int256(_AMT_PER_SEC_MULTIPLIER);
- 1045:             int256 fullCycle = (int256(uint256(_cycleSecs)) * amtPerSec) / amtPerSecMultiplier;
+ 1045:             int256 fullCycle = (int256(uint256(_cycleSecs)) * amtPerSec) / int256(_AMT_PER_SEC_MULTIPLIER);
  1046:             // slither-disable-next-line weak-prng
- 1047:             int256 nextCycle = (int256(timestamp % _cycleSecs) * amtPerSec) / amtPerSecMultiplier;
+ 1047:             int256 nextCycle = (int256(timestamp % _cycleSecs) * amtPerSec) / int256(_AMT_PER_SEC_MULTIPLIER);
  1048:             AmtDelta storage amtDelta = amtDeltas[_cycleOf(uint32(timestamp))];
  1049:             // Any over- or under-flows are fine, they're guaranteed to be fixed by a matching
  1050:             // under- or over-flow from the other call to `_addDelta` made by `_addDeltaRange`.
  1051:             // This is because the total balance of `Drips` can never exceed `type(int128).max`,
  1052:             // so in the end no amtDelta can have delta higher than `type(int128).max`.
  1053:             amtDelta.thisCycle += int128(fullCycle - nextCycle);
  1054:             amtDelta.nextCycle += int128(nextCycle);
  1055:         }
  1056:     }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1036-L1056

**before;**
```js
| test/Drips.t.sol:DripsTest contract |                 |       |        |        |         |
|-------------------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost                     | Deployment Size |       |        |        |         |
| 9220438                             | 44913           |       |        |        |         |
| Function Name                       | min             | avg   | median | max    | # calls |
| balanceAtExternal                   | 1589            | 1892  | 1892   | 2195   | 2       |
| setDripsExternal                    | 2577            | 24085 | 6519   | 170082 | 9       |
| squeezeDripsExternal                | 2859            | 3358  | 3358   | 3857   | 2       |
| squeezeDripsResultExternal          | 2812            | 3311  | 3311   | 3810   | 2       |
```

**after;**
```js
| test/Drips.t.sol:DripsTest contract |                 |       |        |        |         |
|-------------------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost                     | Deployment Size |       |        |        |         |
| 9212411                             | 44866           |       |        |        |         |
| Function Name                       | min             | avg   | median | max    | # calls |
| balanceAtExternal                   | 1589            | 1892  | 1892   | 2195   | 2       |
| setDripsExternal                    | 2577            | 24085 | 6519   | 170082 | 9       |
| squeezeDripsExternal                | 2859            | 3358  | 3358   | 3857   | 2       |
| squeezeDripsResultExternal          | 2812            | 3311  | 3311   | 3810   | 2       |
```

### [G-04] Setting a loop interval instead of a "while" loop causes gas saved

`Drips.sol#L716-L729` has a while loop on this line.
When using `while` loops, the Solidity compiler is aware that a condition needs to be checked first, before executing the code inside the loop.

Firstly, the conditional part is evaluated:

a) if the condition evaluates to true , the instructions inside the loop (defined inside the brackets { ... }) get executed.

b) once the code reaches the end of the loop body (= the closing curly brace } , it will jump back up to the conditional part).

c) the code inside the while loop body will keep executing over and over again until the conditional part turns false.

For loops are more associated with iterations. While loop instead are more associated with repetitive tasks. Such tasks could potentially be infinite if the while loop body never affects the condition initially checked.
Therefore, they should be used with care.


At very small intervals, a large number of loops can be accessed by someone attempting a DOS attack.
This causes very high gas.

```solidity
src/Drips.sol:
  716:             while (true) {
  717:                 uint256 end = (enoughEnd + notEnoughEnd) / 2;
  718:                 if (end == enoughEnd) {
  719:                     return uint32(end);
  720:                 }
  721:                 if (_isBalanceEnough(balance, configs, configsLen, end)) {
  722:                     enoughEnd = end;
  723:                 } else {
  724:                     notEnoughEnd = end;
  725:                 }
  726:             }
  727:             // slither-disable-end incorrect-equality,timestamp
  728:         }
  729      }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L716-L729


**Recommendation:**
Set a reasonable while loop interval

### [G-05] Gas overflow during iteration (DoS)

Each iteration of the cycle requires a gas flow. A moment may come when more gas is required than it is allocated to record one block. In this case, all iterations of the loop will fail.

So set a reasonable max.length

```diff
src/Caller.sol:
  193:     function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {
  194:         returnData = new bytes[](calls.length);
  195:         address sender = _msgSender();
+                 require(calls.length < maxLengt, "max length");
  196:         for (uint256 i = 0; i < calls.length; i++) {
  197:             Call memory call = calls[i];
  198:             returnData[i] = _call(sender, call.to, call.data, call.value);
  199:         }
  200:     }

```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193-L200

### [G-06] Using both `mint` and `safeMint` method at the same time is gas wasted method

For all battle-tested projects, either `mint` or `safeMint` patterns are preferred, this is a design decision; It is evaluated in terms of security concerns and gas optimization, but using both at the same time causes confusion, one of them should be chosen as best practice.


In case of using `mint`, if msg.sender is contract, a faulty contract may be sent without supporting NFT purchase.

In the design, `mint` gas with gas optimization should be preferred.

```solidity
solidity
src/NFTDriver.sol:
  62:     function mint(address to, UserMetadata[] calldata userMetadata)
  63:         public
  64:         whenNotPaused
  65:         returns (uint256 tokenId)
  66:     {
  67:         tokenId = _registerTokenId();
  68:         _mint(to, tokenId);
  69:         if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
  70:     }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L62-L70


```diff
src/NFTDriver.sol:
- 79:     function safeMint(address to, UserMetadata[] calldata userMetadata)
- 80:         public
- 81:         whenNotPaused
- 82:         returns (uint256 tokenId)
- 83:     {
- 84:         tokenId = _registerTokenId();
- 85:         _safeMint(to, tokenId);
- 86:         if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
- 87:     }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L79-L87


### [G-07] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a ``memory`` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional ``MLOAD`` rather than a cheap stack read. Instead of declearing the variable with the ``memory`` keyword, declaring the variable with the ``storage`` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a ``memory`` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires ``memory``, or if the array/struct is being read from another ``memory`` array/struct

5 results - 2 files:
```solidity
src\Caller.sol:

  197:    Call memory call = calls[i];
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L197

```solidity
src\Drips.sol:

  451:    DripsHistory memory drips = dripsHistory[i];

  564:    DripsReceiver memory receiver = receivers[i];

  665:    DripsReceiver memory receiver = newReceivers[i];

  778:    DripsReceiver memory receiver = receivers[i];
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L451


### [G-08] Remove unused codes

Unused code will increase deployment cost.

```diff
src/DripsHub.sol:

  109:     constructor(uint32 cycleSecs)
  110:         Drips(cycleSecs, _erc1967Slot("eip1967.drips.storage"))
  111:         Splits(_erc1967Slot("eip1967.splits.storage"))
  112:     {
- 113:         return;
  114:     }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L113


### [G-09] Using ``delete`` instead of setting  struct to 0 saves gas

2 results - 1 file:

```solidity
src\Splits.sol:
  156:         balance.splittable = 0;

  188:         balance.collectable = 0;
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L156
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L188


**Recommendation Code:**
```diff
src\Splits.sol:

- 156:         balance.splittable = 0;
+ 156:         delete balance.splittable;

```


### [G-10] Not using the named return variables when a function returns, wastes deployment gas


```diff
src\Drips.sol:
   985:     function _dripsRange(
   
-   991:     ) private pure returns (uint32 start, uint32 end_) {
+   991:     ) private pure returns (uint32, uint32) {
   
   1012:         return (start, uint32(end));
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L991


### [G-11] Don't use _msgSender() if not supporting EIP-2771

Use ``msg.sender`` if the code does not implement EIP-2771 trusted forwarder support.

Reference: https://eips.ethereum.org/EIPS/eip-2771

10 results - 3 files:
```solidity
src\AddressDriver.sol:

   47:    return calcUserId(_msgSender());

  171:    erc20.safeTransferFrom(_msgSender(), address(this), amt);
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L47

```solidty
src\Caller.sol:

  107:    address sender = _msgSender();

  115:    address sender = _msgSender();

  149:    require(isAuthorized(sender, _msgSender()), "Not authorized");

  195:    address sender = _msgSender();
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L107


```solidity
src\NFTDriver.sol:

   42:    _isApprovedOrOwner(_msgSender(), tokenId),

  286:    erc20.safeTransferFrom(_msgSender(), address(this), amt);

  295:    return ERC2771Context._msgSender();
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L42


### [G-12] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

5 results - 1 file:
```solidity
src\Drips.sol:
  700:    if (hint1 > enoughEnd && hint1 < notEnoughEnd) {

  708:    if (hint2 > enoughEnd && hint2 < notEnoughEnd) {
  
  898     if (
  899:         pickCurr && pickNew
  900:             && (
  901                 currRecv.userId != newRecv.userId
  902                     || currRecv.config.amtPerSec() != newRecv.config.amtPerSec()
  903             )

  909:    if (pickCurr && pickNew) {

  929:    if (currStartCycle > newStartCycle && state.nextReceivableCycle > newStartCycle) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L652


**Recomendation Code:**
```diff

src\Drips.sol:
- 700:    if (hint1 > enoughEnd && hint1 < notEnoughEnd) {
+     if (hint1 > enoughEnd) {
+         if (hint1 < notEnoughEnd) {
+               }                                   
+           }                                     
```

### [G-13] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html 

Use a larger size then downcast where needed.

29 results - 5 fles:
```solidity
src\Drips.sol:

  /// @audit uint32 receivableCycles
  283:    receivableCycles = toCycle - fromCycle - maxCycles;

  /// @audit uint32 toCycle
  284:    toCycle -= receivableCycles;

  /// @audit uint128 amtPerCycle
  288:    amtPerCycle += state.amtDeltas[cycle].thisCycle;

  /// @audit uint128 receivedAmt
  289:    receivedAmt += uint128(amtPerCycle);

  /// @audit uint128 amtPerCycle
  290:    amtPerCycle += state.amtDeltas[cycle].nextCycle;

  /// @audit uint32 fromCycle, uint32 toCycle
  306:    return toCycle - fromCycle;

  /// @audit uint32 fromCycle
  319:    fromCycle = _dripsStorage().states[assetId][userId].nextReceivableCycle;

  /// @audit uint32 toCycle
  320:    toCycle = _cycleOf(_currTimestamp());

  /// @audit uint128 amt
  429:    amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);

  /// @audit uint32 squeezeEndCap
  432:    squeezeEndCap = drips.updateTime;

  /// @audit uint128 balance
  572:    balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

  /// @audit uint128 mewbalance
  636:    newBalance = uint128(currBalance + realBalanceDelta);

  /// @audit uint32 newMaxEnd 
  637:    newMaxEnd = _calcMaxEnd(newBalance, newReceivers, maxEndHint1, maxEndHint2);

  /// @audit uint32 start
  992:    start = receiver.config.start();

  /// @audit uint32 timestamp
  1122:   return timestamp / _cycleSecs + 1;

  /// @audit uint32 currTimestamp
  1137:   return currTimestamp - (currTimestamp % _cycleSecs);
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283-L284

```solidity
src\DripsHub.sol:

  /// @audit uint32 driverId
  136:    driverId = dripsHubStorage.nextDriverId++;

  /// @audit uint128 receivedAmt
  244:    receivedAmt = Drips._receiveDrips(userId, assetId, maxCycles);

  /// @audit uint128 amt
  278:    amt = Drips._squeezeDrips(userId, assetId, senderId, historyHash, dripsHistory);

  /// @audit uint128 amt
  392:    amt = Splits._collect(userId, _assetId(erc20));
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L136

```solidity
src\Splits.sol:

  /// @audit uint32 splitsWeight
  128:    splitsWeight += currReceivers[i].weight;

  /// @audit uint128 splitAmt
  130:    splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);

  /// @audit uint128 collectableAmt
  131:    collectableAmt = amount - splitAmt;

  /// @audit uint32 splitsWeight
  159:    splitsWeight += currReceivers[i].weight;

  /// @audit uint128 splitAmt
  162:    splitAmt += currSplitAmt;

  /// @audit uint128 collectableAmt
  167:    collectableAmt -= splitAmt;

  /// @audit uint64 totalWeight
  235:    totalWeight += weight;
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L128

```solidity
src\AddressDriver.sol:

  /// @audit uint128 amt
  61:     amt = dripsHub.collect(callerUserId(), erc20);
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L61

```solidity
src\NFTDriver.sol:

  /// @audit uint128 amt
  115:    amt = dripsHub.collect(tokenId, erc20);
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L115


### [G-14] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
5 results - 2 files:
```solidity
src\Drips.sol:
  322:    if (fromCycle == 0 || toCycle < fromCycle) {

  691:    if (configsLen == 0 || balance == 0) {
  
  898:    if (
  899       pickCurr && pickNew
  900            && (
  901                currRecv.userId != newRecv.userId
  902                    || currRecv.config.amtPerSec() != newRecv.config.amtPerSec()

  951:    if (state.nextReceivableCycle == 0 || state.nextReceivableCycle > startCycle) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L322

```solidity
src\Managed.sol:
  53          require(
  54:             admin() == msg.sender || isPauser(msg.sender), "Caller is not the admin or a pauser«
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L53-L54


### [G-15] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

8 results - 7 files:
```solidty
src\AddressDriver.sol:

  30:     constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30

```solidity
src\Drips.sol:

  219:     constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219

```solidity
src\DripsHub.sol:

  109:     constructor(uint32 cycleSecs_)
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109

```solidty
src\ImmutableSplitsDriver.sol:

  28:     constructor(DripsHub _dripsHub, uint32 _driverId) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28

```solidty
src\Managed.sol:

   73:     constructor() {

  158:     constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0)) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L73

```solidity
src\NFTDriver.sol:

  32:     constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32

```solidity
src\Splits.sol:

  94:     constructor(bytes32 splitsStorageSlot) {
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94


**Recommendation:**
Set the constructor to ```payable```


### [G-16] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Context:** 
All Contracts


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ```Drips.sol ``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

Drips.sol function names can be named and sorted according to METHOD ID

```js
Sighash   |   Function Signature
========================
796321aa  =>  create(uint32,uint160,uint32,uint32)
3a859e28  =>  dripId(DripsConfig)
36574f3d  =>  amtPerSec(DripsConfig)
27e07d3c  =>  start(DripsConfig)
4319a265  =>  duration(DripsConfig)
93c5ef21  =>  lt(DripsConfig,DripsConfig)
5025ce09  =>  _receiveDrips(uint256,uint256,uint32)
208536b3  =>  _receiveDripsResult(uint256,uint256,uint32)
f094c2e2  =>  _receivableDripsCycles(uint256,uint256)
65bcd9c1  =>  _receivableDripsCyclesRange(uint256,uint256)
2007ed7a  =>  _squeezeDrips(uint256,uint256,uint256,bytes32,DripsHistory[])
3a028b2f  =>  _squeezeDripsResult(uint256,uint256,uint256,bytes32,DripsHistory[])
8df20ef7  =>  _verifyDripsHistory(bytes32,DripsHistory[],bytes32)
cde22657  =>  _squeezedAmt(uint256,DripsHistory,uint32,uint32)
c1c00721  =>  _dripsState(uint256,uint256)
490762d9  =>  _balanceAt(uint256,uint256,DripsReceiver[],uint32)
5d544393  =>  _balanceAt(uint128,uint32,uint32,DripsReceiver[],uint32)
5ddcd5ab  =>  _setDrips(uint256,uint256,DripsReceiver[],int128,DripsReceiver[],uint32,uint32)
3030aefd  =>  _calcMaxEnd(uint128,DripsReceiver[],uint32,uint32)
fa0fae1f  =>  _isBalanceEnough(uint256,uint256[],uint256,uint256)
2968a833  =>  _buildConfigs(DripsReceiver[])
f41962fa  =>  _addConfig(uint256[],uint256,DripsReceiver)
a665230e  =>  _getConfig(uint256[],uint256)
846d07be  =>  _hashDrips(DripsReceiver[])
ded0b165  =>  _hashDripsHistory(bytes32,bytes32,uint32,uint32)
c4410956  =>  _updateReceiverStates(mapping(uint256)
2b9a67af  =>  _dripsRangeInFuture(DripsReceiver,uint32,uint32)
4d49c72f  =>  _dripsRange(DripsReceiver,uint32,uint32,uint32,uint32)
f2763cf7  =>  _addDeltaRange(DripsState,uint32,uint32,int256)
c5fda91e  =>  _addDelta(mapping(uint32)
abbbd536  =>  _isOrdered(DripsReceiver,DripsReceiver)
058efd18  =>  _drippedAmt(uint256,uint256,uint256)
bee3d77b  =>  _cycleOf(uint32)
25f58251  =>  _currTimestamp()
8e001409  =>  _currCycleStart()
9ecda414  =>  _dripsStorage()
```

### [G-17] Use Minimalist and gas efficient standard ``ERC721`` implementation

Project Use OpenZeppelin ``ERC721`` nft contract.

I recommend using the minimalist and gas-efficient standard ERC1155 implementation.

Reference: https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC721.sol