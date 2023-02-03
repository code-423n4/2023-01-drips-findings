## Summary

**Gas Optimizations**

|  | Issue | Instances |
| --- | --- | --- |
| [G‑01] | ImmutableSplitsDriver.createSplits(): dripsHub  should get cached in local storage | 2 |
| [G‑02] | No need for type conversion as amt is already an uint128 | 1 |
| [G‑03] | Using storage to declare Struct variable inside function | 7 |
| [G-04] | Using delete statement can save gas | 1 |
| [G-05] | Use assembly to write address storage values | 3 |
| [G-06] | Drips.sol: totalDebtPortion._squeezeDripsResult() :  _currCycleStart() should get cached in local storage | 1 |

Total: 15 instances over 6 issues

### **[G-01]**`ImmutableSplitsDriver.createSplits(): dripsHub`  should get cached **in local storage**

See `@audit` tags:

```solidity
/src/ImmutableSplitsDriver.sol
65: emit CreatedSplits(userId, dripsHub.hashSplits(receivers)); //@audit gas: SLOAD 1 dripsHub
66:     dripsHub.setSplits(userId, receivers);  //@audit gas: SLOAD 2 dripsHub
67:     if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata); //@audit gas: SLOAD 3 dripsHub
68:  }
```

- [ImmutableSplitsDriver.sol:65](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L65)

`DripsHub.sol.setDrips(**): realBalanceDelta` should get cached in local storage**

See `@audit` tags:

```solidity
/src/DripsHub.sol
532: if (realBalanceDelta > 0) {  //@audit gas: SLOAD 1 realBalanceDelta
533:            erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta)); //@audit gas: SLOAD 2 realBalanceDelta
534:        } else if (realBalanceDelta < 0) {
535:            _decreaseTotalBalance(erc20, uint128(-realBalanceDelta)); //@audit gas: SLOAD 3 realBalanceDelta
536:            erc20.safeTransfer(msg.sender, uint128(-realBalanceDelta)); //@audit gas: SLOAD 4 realBalanceDelta

```

- [DripsHub.sol:532](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L532)

### **[G-02] No need for type conversion as `amt` is already an `uint128`**

**Affected source code:**

```diff
/src/Drips.sol
-497:    return uint128(amt);
+497:    return amt;

```

- [Drips.sol:497](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L497)

### **[G-03] Using `storage` to declare Struct variable inside function**

instead of caching variable to memory. read it directly from storage.

*There are 7 instances of this issue:*

**Affected source code:**

```solidity
/SubprotocolRegistry.sol
89: SubprotocolData memory subprotocolData = subprotocols[_name]; 

/src/Drips.sol
451: DripsHistory memory drips = dripsHistory[i];
491: DripsReceiver memory receiver = receivers[idx];
564: DripsReceiver memory receiver = receivers[i];
665: DripsReceiver memory receiver = newReceivers[i];
778: DripsReceiver memory receiver = receivers[i];

/src/Caller.sol
197: Call memory call = calls[i];
```

- [SubprotocolRegistry.sol:89](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L89)
- [Drips.sol:451](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L451)

### **[G-04] Using `delete` statement can save gas**

*There are 1 instances of this issue:*

**Affected source code:**

```solidity
/src/Splits.sol
188: balance.collectable = 0;
```

Mitigation:

```diff
/src/Splits.sol
-188: balance.collectable = 0;
+188: delete balance.collectable;
```

### **[G-05] Use `assembly` to write *address storage values***

*There are 3 instances of this issue:*

**Affected source code:**

```solidity
/src/NFTDriver.sol
32 constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
36: dripsHub = _dripsHub;

/src/AddressDriver.sol
30 constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)  
33: dripsHub = _dripsHub;

/src/ImmutableSplitsDriver.sol
28 constructor(DripsHub _dripsHub, uint32 _driverId) {
29: dripsHub = _dripsHub;
```

### **[G-06]  Drips.sol: `totalDebtPortion._squeezeDripsResult()` :  `_currCycleStart()` should get cached**

- [Drips.sol:416](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L416)

```solidity
/src/Drips.sol
414: currCycleConfigs = 1;
415:            // slither-disable-next-line timestamp
416:            if (sender.updateTime >= _currCycleStart()) currCycleConfigs = sender.currCycleConfigs;
417:        }
418:        squeezedRevIdxs = new uint256[](dripsHistory.length);
419:        uint32[2 ** 32] storage nextSqueezed =
420:            _dripsStorage().states[assetId][userId].nextSqueezed[senderId];
421:        uint32 squeezeEndCap = _currTimestamp();
422:        for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
423:            DripsHistory memory drips = dripsHistory[dripsHistory.length - i];
424:            if (drips.receivers.length != 0) {
425:                uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
426:                if (squeezeStartCap < _currCycleStart()) squeezeStartCap = _currCycleStart();
427:                if (squeezeStartCap < squeezeEndCap) {
428:                    squeezedRevIdxs[squeezedNum++] = i;
429:                    amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);
430:                }
431:            }
```

Mitigation: Cached  `uint32 cycleStart = _currCycleStart();`

```diff
          currCycleConfigs = 1;
            // slither-disable-next-line timestamp
+         uint32 cycleStart = _currCycleStart(); 
          if (sender.updateTime >= cycleStart) currCycleConfigs = sender.currCycleConfigs;  //@audit gas: SLOAD 1  _currCycleStart()
          }
          squeezedRevIdxs = new uint256[](dripsHistory.length);
          uint32[2 ** 32] storage nextSqueezed =
         _dripsStorage().states[assetId][userId].nextSqueezed[senderId];
          uint32 squeezeEndCap = _currTimestamp();
          for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
          DripsHistory memory drips = dripsHistory[dripsHistory.length - i];
          if (drips.receivers.length != 0) {
                uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
                if (squeezeStartCap < cycleStart) squeezeStartCap = cycleStart;  //@audit gas: SLOAD 2   _currCycleStart()
                if (squeezeStartCap < squeezeEndCap) {
                    squeezedRevIdxs[squeezedNum++] = i;
                    amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);
                }
           }
```