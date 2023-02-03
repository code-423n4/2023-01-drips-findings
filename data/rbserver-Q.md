## [01] MALICIOUS USER, WHO OWNS SPLITTABLE FUNDS, CAN CALL `DripsHub.setSplits` FUNCTION TO FRONTRUN OTHER USER'S `DripsHub.split` FUNCTION CALL, WHICH CAN BREAK AGREEMENT BETWEEN THESE USERS
Based on the current implementation, the following `DripsHub.split` function is callable by anyone. This function's comment implies that the user, who owns the splittable funds, can call the `DripsHub.setSplits` function to frontrun the other user's `DripsHub.split` function call so the other user, who is one of the old splits receivers but not one of the new, cannot receive the corresponding funds. Here, the user, who owns the splittable funds, is assumed to be trusted. However, such user can become malicious in reality. For example, Alice, who owns the splittable funds, previously agrees to distribute some funds to Bob. Later, Alice becomes malicious and breaks the agreement without notifying Bob. After noticing that Bob has called the `DripsHub.split` function, Alice frontruns it by calling the `DripsHub.setSplits` function to remove Bob from the splits receivers. After the frontrunning, Bob's `DripsHub.split` function call does not distribute any funds to him so he loses the funds that he is entitled to under his and Alice's previous agreement.

As a mitigation, if the `DripsHub.split` function needs to remain callable by anyone, flashbots can be used to keep users' `DripsHub.split` transactions away from the mempool for counteracting frontrunning.

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L336-L361
```solidity
    ...
    /// Because the user can update their splits configuration at any time,
    /// it is possible that calling this function will be frontrun,
    /// and all the splittable funds will become splittable only using the new configuration.
    /// The user must be trusted with how funds sent to them will be splits,
    /// in the end they can do with their funds whatever they want by changing the configuration.
    ...
    function split(uint256 userId, IERC20 erc20, SplitsReceiver[] memory currReceivers)
        public
        whenNotPaused
        returns (uint128 collectableAmt, uint128 splitAmt)
    {
        return Splits._split(userId, _assetId(erc20), currReceivers);
    }
```

## [02] MISSING TWO-STEP PROCEDURES FOR CHANGING ADMIN AND DRIVER ADDRESS
Calling the following `Managed.changeAdmin` function changes the admin to `newAdmin`. Because this function does not include a two-step procedure, it is possible that the admin is changed to a wrong address. When an unknown and untrusted address becomes the admin, important functions like `Managed.grantPauser`, `Managed.revokePauser`, `Managed.pause`, and `Managed.unpause` can become no-ops or be maliciously called, which can break the protocol. For example, after the admin is changed to a wrong address due to a careless `Managed.changeAdmin` function call, the owner of this wrong address can call the `Managed.revokePauser` function to revoke all other pausers and then the `Managed.pause` function to pause the protocol; as a result, all funds that are sent to the protocol cannot be collected and transferred out.

https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84-L86
```solidity
    function changeAdmin(address newAdmin) public onlyAdmin {
        _changeAdmin(newAdmin);
    }
```

Similarly, the following `DripsHub.updateDriverAddress` function does not include a two-step procedure so `_dripsHubStorage().driverAddresses[driverId]` can possibly be changed to a wrong address. As a result, the functions associated with the affected `driverId` can become no-ops or be maliciously called. For example, after `_dripsHubStorage().driverAddresses[driverId]` is changed to a wrong address because of a careless `DripsHub.updateDriverAddress` function call, if this wrong address corresponds to a malicious contract, the owner of such contract can use it to call the `DripsHub.collect` function with a valid `userId` to withdraw funds that actually should belong to the user corresponding to this `userId`.

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L152-L156
```solidity
    function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
        _assertCallerIsDriver(driverId);
        _dripsHubStorage().driverAddresses[driverId] = newDriverAddr;
        emit DriverAddressUpdated(driverId, msg.sender, newDriverAddr);
    }
```

As mitigations, please consider implementing two-step procedures for changing the admin and driver address.

## [03] EXTRA ETH AMOUNT SENT WHEN CALLING `Caller.callBatched` FUNCTION IS LOCKED IN `Caller` CONTRACT, WHICH IS RISKY TO USERS WHO DO NOT READ THIS FUNCTION'S CODE
The following `Caller.callBatched` function's comment implies that the extra ETH amount sent when calling this function will be locked in the `Caller` contract. However, this risk can still cause users, especially the non-technical ones who do not read this function's code, to lose the extra ETH amount sent by them. For example, Alice does not read this function's comment and does not know about this risk. When calling this function, she sends 10 ETH but all `call.value` of her `calls` only add up to 9 ETH. In this case, she loses the extra 1 ETH, which is locked in this contract.

As a mitigation, the `Caller.callBatched` function can be updated to check if all `call.value` of the `calls` can add up to `msg.value` or not. If not, calling this function should revert.

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L185-L200
```solidity
    ...
    /// This function is payable, any Ether sent to it can be used in the batched calls.
    /// Any unused Ether will stay in this contract,
    /// anybody will be able to use it in future calls to `callBatched`.
    ...
    function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {
        returnData = new bytes[](calls.length);
        address sender = _msgSender();
        for (uint256 i = 0; i < calls.length; i++) {
            Call memory call = calls[i];
            returnData[i] = _call(sender, call.to, call.data, call.value);
        }
    }
```

## [04] CALLING `DripsHub.registerDriver` FUNCTION WITH MEANINGLESS `driverAddr` INPUT FOR MANY TIMES CAN CAUSE EVENT LOG POISONING AND USE UP `dripsHubStorage.nextDriverId`
An malicious actor can call the following `DripsHub.registerDriver` function with a meaningless `driverAddr` input for many times to emit useless `DriverRegistered` events. This causes event log poisoning in which the monitor systems that consume the `DriverRegistered` event can be confused and spammed. This action can also use up `dripsHubStorage.nextDriverId` and leave less `dripsHubStorage.nextDriverId` for registering valid driver addresses.

As a mitigation, please consider update the `DripsHub.registerDriver` function to be only callable by certain trusted parties, such as the protocol admin.

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L134-L139
```solidity
    function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
        DripsHubStorage storage dripsHubStorage = _dripsHubStorage();
        driverId = dripsHubStorage.nextDriverId++;
        dripsHubStorage.driverAddresses[driverId] = driverAddr;
        emit DriverRegistered(driverId, driverAddr);
    }
```

## [05] SPLITS RECEIVERS MAY NEED TO BE UPDATED BEFORE SPLITTING FUNDS FOR DIFFERENT ERC20 TOKENS, WHICH IS INCONVENIENT
Calling the following `Splits._setSplits` function sets splits receivers for the corresponding `userId` in which each `userId` can only have one list of splits receivers. Yet, because a user can control splittable funds of different ERC20 tokens, it is possible that the user wants to have different splits receivers for different ERC20 tokens, which is not possible currently. If the user wants to split different ERC20 tokens to different receivers, she or he has to update the splits receivers before each `DripsHub.split` function call for different ERC20 tokens, which is inconvenient and costs more gas. To improve the user experience, please consider maintaining different splits receivers for different ERC20 tokens for each `userId`.

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L212-L220
```solidity
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

## [06] MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS
To prevent unintended behaviors, critical address inputs should be checked against `address(0)`.

`address(0)` check is missing for `forwarder` in the following constructor. Please consider checking it.
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30-L35
```solidity
    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
    {
        dripsHub = _dripsHub;
        driverId = _driverId;
    }
```

`address(0)` check is missing for `pauser` in the following function. Please consider checking it.
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L90-L93
```solidity
    function grantPauser(address pauser) public onlyAdmin {
        require(_managedStorage().pausers.add(pauser), "Address already is a pauser");
        emit PauserGranted(pauser, msg.sender);
    }
```

`address(0)` check is missing for `forwarder` in the following constructor. Please consider checking it.
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32-L38
```solidity
    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
        ERC721("DripsHub identity", "DHI")
    {
        dripsHub = _dripsHub;
        driverId = _driverId;
    }
```

## [07] CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS
To improve readability and maintainability, a constant can be used instead of the magic number. Please consider replacing the magic numbers, such as `224`, used in the following code with constants.

```solidity
src\AddressDriver.sol
  41: return (uint256(driverId) << 224) | uint160(userAddr);  

src\Drips.sol
  74: return uint32(DripsConfig.unwrap(config) >> 224);   
  79: return uint160(DripsConfig.unwrap(config) >> 64);   
  84: return uint32(DripsConfig.unwrap(config) >> 32);   

src\ImmutableSplitsDriver.sol
  37: return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_counterSlot).value; 

src\NFTDriver.sol
  51: return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_mintedTokensSlot).value;
```

## [08] `MAX_TOTAL_BALANCE` IS NOT CODED AS MINIMUM OF `_MAX_TOTAL_DRIPS_BALANCE` AND `_MAX_TOTAL_SPLITS_BALANCE`, WHICH DOES NOT MATCH CODE COMMENT
The following code directly sets `MAX_TOTAL_BALANCE` to `_MAX_TOTAL_DRIPS_BALANCE` but its comment indicates that `MAX_TOTAL_BALANCE` should be the minimum of `_MAX_TOTAL_DRIPS_BALANCE` and `_MAX_TOTAL_SPLITS_BALANCE`. Although `_MAX_TOTAL_DRIPS_BALANCE` is smaller than `_MAX_TOTAL_SPLITS_BALANCE` as shown in the code below, `MAX_TOTAL_BALANCE` could still be coded as the minimum of the two constants to match the code comment for better readability and maintainability.

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L68-L70
```solidity
    /// @notice The total amount the contract can store of each token.
    /// It's the minimum of _MAX_TOTAL_DRIPS_BALANCE and _MAX_TOTAL_SPLITS_BALANCE.
    uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L112
```solidity
    uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
```

https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L25
```solidity
    uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;
```

## [09] REDUNDANT NAMED RETURNS
When a function has unused named returns and used return statement, these named returns become redundant. To improve readability and maintainability, these variables for the named returns can be removed while keeping the return statements for the functions associated with the following code.

```solidity
src\Caller.sol
  124: function isAuthorized(address sender, address user) public view returns (bool authorized) { 
  132: function allAuthorized(address sender) public view returns (address[] memory authorized) { 

src\Drips.sol
  303: returns (uint32 cycles) 
  685: ) private view returns (uint32 maxEnd) {    
  837: returns (bytes32 dripsHash) 
  1120: function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) { 
  1128: function _currTimestamp() private view returns (uint32 timestamp) { 

src\DripsHub.sol
  358: returns (uint128 collectableAmt, uint128 splitAmt)  
  642: function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {   

src\Managed.sol
  105: function isPauser(address pauser) public view returns (bool isAddrPauser) { 
  112: function allPausers() public view returns (address[] memory pausersList) { 

src\Splits.sol
  262: function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) { 
  273: returns (bytes32 receiversHash) 
```

## [10] FLOATING PRAGMAS
It is a best practice to lock pragmas instead of using floating pragmas to ensure that contracts are tested and deployed with the intended compiler version. Accidentally deploying contracts with different compiler versions can lead to unexpected risks and undiscovered bugs. Please consider locking pragmas for the following files.

```solidity
src\AddressDriver.sol
  2: pragma solidity ^0.8.17;    

src\Caller.sol
  2: pragma solidity ^0.8.17;    

src\Drips.sol
  2: pragma solidity ^0.8.17;    

src\DripsHub.sol
  2: pragma solidity ^0.8.17;    

src\ImmutableSplitsDriver.sol
  2: pragma solidity ^0.8.17;    

src\Managed.sol
  2: pragma solidity ^0.8.17;    

src\NFTDriver.sol
  2: pragma solidity ^0.8.17;    

src\Splits.sol
  2: pragma solidity ^0.8.17;    
```

## [11] CONFUSING NATSPEC `@param` USAGE
Because `amt` is the returned variable of the following `_drippedAmt` function, `@return` can be used instead of `@param` in the corresponding NatSpec comment to avoid confusion.

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1092-L1115
```solidity
    /// @param amt The dripped amount
    function _drippedAmt(uint256 amtPerSec, uint256 start, uint256 end)
        private
        view
        returns (uint256 amt)
    {
        ...
    }
```

## [12] INCOMPLETE NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss the `@param` and/or `@return` comments. Please consider completing the NatSpec comments for these functions.

```solidity
src\Drips.sol
  985: function _dripsRange(   

src\Managed.sol
  78: function admin() public view returns (address) {    
  117: function paused() public view returns (bool isPaused) { 
```

## [13] MISSING NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss NatSpec comments. Please consider adding NatSpec comments for these functions.

```solidity
src\AddressDriver.sol
  170: function _transferFromCaller(IERC20 erc20, uint128 amt) internal {  

src\Caller.sol
  202: function _call(address sender, address to, bytes memory data, uint256 value)    

src\DripsHub.sol
  124: function _assertCallerIsDriver(uint32 driverId) internal view { 
  629: function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {    
  635: function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {    

src\NFTDriver.sol
  285: function _transferFromCaller(IERC20 erc20, uint128 amt) internal {  

src\Splits.sol
  98: function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal {
```