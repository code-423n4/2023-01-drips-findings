**Drips.sol**
- L108/112 - The constants _AMT_PER_SEC_EXTRA_DECIMALS and _MAX_TOTAL_DRIPS_BALANCE are created but never used, therefore they should be removed.

- L1122 - It is not clear which operation is executed first (timestamp / _cycleSecs + 1). Here a problem could be generated, since _cycleSecs is set in the constructor and it could be 0, generating an exception.
But it is not clear if it is going to be executed like this: (timestamp / _cycleSecs) + 1 or like this_ timestamp / (_cycleSecs + 1)
 

**DripsHub.sol**
- L4 - The library and struct: DripsConfigy and DripsConfigImpl are imported but they are never used, therefore they should be removed.

- L56/58/60/62 - The constants MAX_DRIPS_RECEIVERS, AMT_PER_SEC_EXTRA_DECIMALS, AMT_PER_SEC_MULTIPLIER and MAX_SPLITS_RECEIVERS are created but never used, therefore they should be removed.


**Splits.sol**
- L25 - The constant _MAX_TOTAL_SPLITS_BALANCE is created, but it is never used, therefore it should be removed.


**NFTDriver.sol**
- L4 - The struct: DripsHistory is imported but it is never used, therefore it should be removed.

- L12 - The IERC721 interface is imported, but it is not necessary, since ERC721 already imports that interface.


**Managed.sol**
- L6 - The contract is imported: StorageSlot but it is never used, therefore it should be deleted.


**AddressDriver.sol**
- L5 - The struct: DripsHistory is imported but it is never used, therefore it should be deleted.
