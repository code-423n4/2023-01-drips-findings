# [L-01] Remove redundant imports
File Addressаriver.sol: 
line 5: `DripsHistory`

File DripsHub.sol: 
line 4: `DripsConfig`, `DripsConfigImpl`

File NFTDriver.sol: 
line 4: `DripsHistory`


# [L-02] Use external modifier instead of public
Use external modifier instead of public if the function is not used for internal calls
File DripsHub.sol: 
line 134: `function registerDriver(address driverAddr)`
line 152: `function updateDriverAddress(uint32 driverId, address newDriverAddr)`
line 196: `function receivableDripsCycles(uint256 userId, IERC20 erc20)`
line 297: `function squeezeDripsResult`
line 328: `function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)`
line 432: `function dripsState(uint256 userId, IERC20 erc20)`
line 460: `balanceAt`
line 576: ` function setSplits(uint256 userId, SplitsReceiver[] memory receivers)`
line 608: `function emitUserMetadata(uint256 userId, UserMetadata[] calldata userMetadata)`

# [L-03] Not unused returned variable
In some place is added naming of returned data but is not used directly
File DripsHub.sol: 
line 642: `function _assetId(IERC20 erc20) internal pure returns (uint256 assetId)`
line 598: `returns (bytes32 receiversHash)`
line 587: `function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash)`
line 562: `public pure returns (bytes32 dripsHistoryHash) `
line 546:  ` function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash)`
line 465: `public view returns (uint128 balance)`
line 436-440: 
```     
bytes32 dripsHash,
bytes32 dripsHistoryHash,
uint32 updateTime,
uint128 balance,
uint32 maxEnd
```
line 362: `function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt)`
line 358: ` returns (uint128 collectableAmt, uint128 splitAmt)`
line 331: `returns (uint128 collectableAmt, uint128 splitAmt)`
line 317: `function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt)`
line 199: `returns (uint32 cycles)`
line 181: `function totalBalance(IERC20 erc20)`
line 169: ` function cycleSecs() public view returns (uint32 cycleSecs_)`
### Recomended:
Use the variable or just remove him.