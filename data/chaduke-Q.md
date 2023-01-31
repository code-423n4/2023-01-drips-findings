QA3. It is advised to follow the common practice: to choose a slot that has no KNOWN preimage. It is much easier to find a second preimage if we already know the first preimage. 
 
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L72

Mitigation: 
Add -1 to avoid known preimage. 

```
bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage") - 1;
```

QA4. If a driver set the wrong (possibly inaccessible) ``newDriverAddr``, then he can will lose his driver's privilege forever. 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152-L156
```
 function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
        _assertCallerIsDriver(driverId);
        _dripsHubStorage().driverAddresses[driverId] = newDriverAddr;
        emit DriverAddressUpdated(driverId, msg.sender, newDriverAddr);
    }
```

Mitigation: This is similar to the change of the owner of a contract. We need to  follow a two-step procedure: 1) first set the new pending address; 2) use the new pending address to accept the change to the final new driver address. 

QA5. It is important to emit events when critical state variables are changed by some functions. 

a) https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L629-L637



QA6. [_addDeltaRange()](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1020-L1030) fails to check the input range is valid. 
One caller [_updateReceiverStates()](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L872-L965) forgot to check this and possibly pass invalid ranges to this function, causing a chaos to the system.

1) The following code does not check that ``start <= end`` and even when ``start > end``, it still performs the update. 
```javascript
function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)
        private
    {
        // slither-disable-next-line incorrect-equality,timestamp
        if (start == end) {
            return;
        }
        mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
        _addDelta(amtDeltas, start, amtPerSec);
        _addDelta(amtDeltas, end, -amtPerSec);
    }
```

2) ``_updateReceiverStates()`` calls ``_addDeltaRange()`` in the [following code](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L919-L920):
```javascript
                     _addDeltaRange(state, currStart, newStart, -amtPerSec);
                    _addDeltaRange(state, currEnd, newEnd, amtPerSec);
```
3) It is possible that (currStart > newStart) and (currEnd > newEnd). In this case, the ``_updateReceiverStates()`` will make the wrong updates. The above code has its own other problem (wrong range applied), but here we illustates another problem: when both the caller and callee does not perform a valid range check, there is a bad consequence.

4) This scenario shows the importance of doing a valid range check for ``_addDeltaRange()``.

Mitigation:
We add a valid range check below:
```diff
function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)
        private
    {
+       if(start > end) revert InvalidRange();

        // slither-disable-next-line incorrect-equality,timestamp
        if (start == end) {
            return;
        }
        mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
        _addDelta(amtDeltas, start, amtPerSec);
        _addDelta(amtDeltas, end, -amtPerSec);
    }
```

QA7. False emit is possible here. 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L215
```
function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {
        SplitsState storage state = _splitsStorage().splitsStates[userId];
        bytes32 newSplitsHash = _hashSplits(receivers);
        emit SplitsSet(userId, newSplitsHash);
        if (newSplitsHash != state.splitsHash) {
            _assertSplitsValid(receivers, newSplitsHash);
            state.splitsHash = newSplitsHash;
        }
    }
```
We need to check whether it is a new receiver list. The emit statement needs to be in the if block.
```diff
function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {
        SplitsState storage state = _splitsStorage().splitsStates[userId];
        bytes32 newSplitsHash = _hashSplits(receivers);
-        emit SplitsSet(userId, newSplitsHash);
        if (newSplitsHash != state.splitsHash) {
            _assertSplitsValid(receivers, newSplitsHash);
            state.splitsHash = newSplitsHash;
+          emit SplitsSet(userId, newSplitsHash);
        }
    }
```

