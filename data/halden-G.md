
# [G-01] Use constant variable instead of immutable
Use immutable variables only when they are initilized in constructor.
File DripsHub.sol: 
line 72: `bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage");`
File NFTDriver.sol: 
line 27: ` bytes32 private immutable _mintedTokensSlot = _erc1967Slot("eip1967.nftDriver.storage");`
File Managed.sol
line 20: `bytes32 private immutable _managedStorageSlot = _erc1967Slot("eip1967.managed.storage");`
File ImmutableSplitsDriver.sol
line 19: `    bytes32 private immutable _counterSlot = _erc1967Slot("eip1967.immutableSplitsDriver.storage");`


# [G-02] require() strings longer than 32 bytes cost extra gas
File DripsHub.sol: 
line 125: `require(driverAddress(driverId) == msg.sender, "Callable only by the driver");`
File split.sol
line 244: `require(totalWeight <= _TOTAL_SPLITS_WEIGHT, "Splits weights sum too high");`
line 238: `require(prevUserId != userId, "Duplicate splits receivers");`
line 239: `require(prevUserId < userId, "Splits receivers not sorted by user ID");`
line 234: `require(weight != 0, "Splits receiver weight is zero");`
line 255: ` _hashSplits(currReceivers) == _splitsHash(userId), "Invalid current splits receivers"`

# [G-03] Using unchecked blocks to save gas - Increments in for loop can be unchecked
File DripsHub.sol: 
line 613: `for (uint256 i = 0; i < userMetadata.length; i++)`
File SPlits.sol:
line 127: `for (uint256 i = 0; i < currReceivers.length; i++)`
line 158: `for (uint256 i = 0; i < currReceivers.length; i++)`