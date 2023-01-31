# Summary
|ID     | Finding|  Gas saved |Instances|
|:----: | :---           |  :----:         |:----:         |
|1       |Make for loop unchecked|644|13|
|2       |Use an unchecked block when operands can't underflow/overflow|688|7|
|3       |Write element of storage struct to memory when used more than once|10|1|
|4       |Call block.timestamp direclty instead of function|22|1|
|5       |Make 3 event parameters indexed when possible|1059|2|
|6       |Transfer erc20 immediately to Dripshub|57824|1|
|7       |Transfer ERC20 immediately to the user|22112|1|
|8       |Use double if statements instead of &&|4|40|
|9       |Miscellaneous| 100|3|


# Details
## 1 Make for loop unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow.

- [Caller.sol#L196-L199](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196-L199): `callBatched()` gas saved: 76
- [ImmutableSplitsDriver.sol#L61-L63](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61-L63): `createSplits()` gas saved: 92
- [Drips.sol#L247-L249](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247-L249): `DripsHub: receiveDrips()` gas saved: 58 
- [Drips.sol#L287-L291](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287-L291): `DripsHub: receiveDripsResult()` gas saved: 54
- [Drips.sol#L357-L364](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L357-L364): `DripsHub: squeezeDrips()` gas saved: 32
- [Drips.sol#L450-L459](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450-L459): `DripsHub: squeezeDripsResult()` gas saved: 71
- [Drips.sol#L490-L497](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490-L497): `DripsHub: squeezeDripsResult()` gas saved: 29
- [Drips.sol#L563-L573](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563-L573): `DripsHub: balanceAt()` gas saved: 59
- [Drips.sol#L662-L668](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L662-L668): `DripsHub: setDrips()` gas saved: 31
- [DripsHub.sol#L613-L616](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613-L616): `emitUserMetadata()` gas saved: 59
- [Splits.sol#L127-L129](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127-L129): `DripsHub: splitResult()` gas saved: 10
- [Splits.sol#L158-L166](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158-L166): `DripsHub: split()` gas saved: 10
- [Splits.sol#L231-L243](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231-L243): `DripsHub: setSplits()` gas saved: 63

There are ways to make a for loop unchecked in a safe way
```diff
-       for (uint256 i = 0; i < calls.length; i++) {
-       for (uint256 i = 0; i < calls.length;) {
            Call memory call = calls[i];
            returnData[i] = _call(sender, call.to, call.data, call.value);
+           unchecked{
+              i++
+           } 
        }
```
```diff
+       function unchecked_inc(uint256 x) private pure returns (uint256) {
+          unchecked {
+             return x + 1;
+          }
+       }

-       for (uint256 i = 0; i < calls.length; i++) {
+       for (uint256 i = 0; i < calls.length;  i = unchecked_inc(i)) {
            Call memory call = calls[i];
            returnData[i] = _call(sender, call.to, call.data, call.value);
        }
```
## 2 Use an unchecked block when operands can't underflow/overflow
[Drips.sol#L480](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L480)

Division can't overflow or underflow unless the divisor is -1. Which is not the case here. `DripsHub: squeezeDripsResult()` gas saved: 60
```solidity
    uint256 idxMid = (idx + idxCap) / 2;
```
[Drips.sol#L655](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L655)

There is no possibility of overflowing when incrementing by one. currCycleConfigs is a uint32 but even for that will take a massive amount of time to make it overflow. `DripsHub: setDrips()` gas saved: 215
```solidity
    state.currCycleConfigs++;
```
For ++, the same applies to:
- [Drips.sol#L428](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L428) `DripsHub: squeezeDrips()` gas saved: 28
- [Drips.sol#L959](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L959) and [Drips.sol#L962](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L962): `DripsHub: setDrips()` gas saved: 35
- [DripsHub.sol#L136](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L136) `DripsHub: registerDriver()` gas saved: 200

[DripsHub.sol#L632](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632): because of the require statement above, it can't overflow. `give()` gas saved: 54
[DripsHub.sol#L636](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L636): amount will never be larger than the total balance: `collect()` gas saved: 96
## 3 Write element of storage struct to memory when used more than once
When a struct contains a nested mapping, it's not possible to save it in memory. But it's possible to save one element of the struct to memory when it's used more than once. `DripsHub: balanceAt()` gas saved: 10

- [Drips.sol#L539-L542](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L539-L542)
```diff
        DripsState storage state = _dripsStorage().states[assetId][userId];
+       uint32 updateTime = state.updateTime;
-       require(timestamp >= state.updateTime, "Timestamp before last drips update");
+       require(timestamp >= updateTime, "Timestamp before last drips update");
        require(_hashDrips(receivers) == state.dripsHash, "Invalid current drips list");
-       return _balanceAt(state.balance, state.updateTime, state.maxEnd, receivers, timestamp);
+       return _balanceAt(state.balance, updateTime, state.maxEnd, receivers, timestamp);
```
## 4 Call block.timestamp direclty instead of function
The `_currTimestamp()` function casts the `block.timestamp` to a uint32. However it's not always necessary to have a uint32.

In the example below you are assigning the timestamp to a uint256. Which makes it unnecessary to cast the timestamp to uint32. So it's better to call `block.timestamp` directly.

[Drips.sol#L689](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L689): `DripsHub: setDrips()` gas saved: 22
```diff
-    uint256 enoughEnd = _currTimestamp();
+    uint256 enoughEnd = block.timestamp;
```
## 5 Make 3 event parameters indexed when possible
It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

- [DripsHub.sol#L93](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L93)
```diff
-    event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);
+    event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes indexed value);
```
The event is used in the function emitUserMetaData, this function is used multiple times. The gas saved for making all the parameters indexed is:
- DripsHub.sol: `emitUserMetadata()` gas saved: 364
- AddressDriver.sol: `emitUserMetadata()` gas saved: 208
- ImmutableSplitsDriver.sol: `createSplits()` gas saved: 139
- NFTDriver.sol: `emitUserMetadata()` gas saved: 139
- NFTDriver.sol: `safeMint()` gas saved: 209

The same extra indexed parameter can be applied to:
- [Drips.sol#L153](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L153) `DripsHub: setDrips()`
## 6 Transfer erc20 immediately to Dripshub
- `Dripshub: give()` gas saved: 9449
- `AddressDriver: give()` gas saved: 29439
- `NFTDriver: give()` gas saved: 18936

When you call the `give()` function in the Address or NFTDriver. The erc20 token are first getting send to those contracts and afterwards to the DripsHub contract. It's also possible to send the tokens directly to the dripsHub contract.

[AddressDriver.sol#L170-L176](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L170-L176) [NFTDriver.sol#L285-L291](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L285-L291)
```diff
    function _transferFromCaller(IERC20 erc20, uint128 amt) internal {
-       erc20.safeTransferFrom(_msgSender(), address(this), amt);
+       erc20.safeTransferFrom(_msgSender(), address(dripsHub), amt);
        // Approval is done only on the first usage of the ERC-20 token in DripsHub by the driver
-       if (erc20.allowance(address(this), address(dripsHub)) == 0) {
-           erc20.safeApprove(address(dripsHub), type(uint256).max);
        }
    }
```
[DripsHub.sol#L417](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L417)
```diff
    function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)
        public
        whenNotPaused
        onlyDriver(userId)
    {
        _increaseTotalBalance(erc20, amt);
        Splits._give(userId, receiver, _assetId(erc20), amt);
-       erc20.safeTransferFrom(msg.sender, address(this), amt);
    }
```
setDrips need to be adjusted or you can create a seperate function for that.
## 7 Transfer ERC20 immediately to the user
- `AddressDriver: collect()` gas saved: 12954
- `NFTDriver: collect()` gas saved: 9158

The same thing can be done for the `collect()` function. Instead of transferring first to the address or NFTdriver. You can instantly transfer to the user.
[DripsHub.sol#L386-L395)](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L386-L395)
```diff
-   function collect(uint256 userId, IERC20 erc20)
+   function collect(uint256 userId, IERC20 erc20, address transferTo)
        public
        whenNotPaused
        onlyDriver(userId)
        returns (uint128 amt)
    {
        amt = Splits._collect(userId, _assetId(erc20));
        _decreaseTotalBalance(erc20, amt);
-       erc20.safeTransfer(msg.sender, amt);
+       erc20.safeTransfer(transferTo, amt);
    }
```
[AddressDriver.sol#L60-L63](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L60-L63)
```diff
    function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
-       amt = dripsHub.collect(callerUserId(), erc20);
+       amt = dripsHub.collect(callerUserId(), erc20, transferTo);
-       erc20.safeTransfer(transferTo, amt);
    }
```
## 8 Use double if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

- [Drips.sol#L700](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L700)
- [Drips.sol#L709](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L709)
- [Drips.sol#L909](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L909)
- [Drips.sol#L929](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L929)
## Miscellaneous
### Don't call a function when initializing an immutable variable
Saves a little bit of deployment gas
```diff
-   bytes32 private immutable _counterSlot = _erc1967Slot("eip1967.immutableSplitsDriver.storage");
+   bytes32 private immutable _counterSlot = bytes32(uint256(keccak256(bytes("eip1967.immutableSplitsDriver.storage"))) - 1);
```
### Use a mapping type of a struct directly instead of assigning it to another storage variable
- [Drips.sol#L246-L254](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L246-L254)
```diff
-   mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
    for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
-       delete amtDeltas[cycle];
+       delete state.amtDeltas[cycle];
    }
    // The next cycle delta must be relative to the last received cycle, which got zeroed.
    // In other words the next cycle delta must be an absolute value.
    if (finalAmtPerCycle != 0) {
-       amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
+       state.amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
    }
```
### If statement can be adjusted
In the `_receiveDrips()` function you can change the check to `receivedAmt != 0`. Because when fromCycle and toCycle are the same. ReceivedAmt will be 0.

[Drips.sol#L243](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L243) `receiveDrips()` gas saved: 75
```diff
        (receivedAmt, receivableCycles, fromCycle, toCycle, finalAmtPerCycle) =
            _receiveDripsResult(userId, assetId, maxCycles);
-       if (fromCycle != toCycle) {
+       if (receivedAmt != 0) {
```