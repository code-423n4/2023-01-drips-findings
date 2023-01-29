QA3. It is advised to follow the common practice: to choose a slot that has no KNOWN preimage. It is much easier to find a second preimage if we already know the first preimage. 
 
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L72

Mitigation: 
Add -1 to avoid known preimage. 

```
bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage") - 1;
```
