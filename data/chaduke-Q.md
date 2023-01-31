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

