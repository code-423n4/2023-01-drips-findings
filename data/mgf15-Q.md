
ImmutableSplitsDriver.sol#53

 function createSplits(SplitsReceiver[] calldata receivers, UserMetadata[] calldata userMetadata)
        public
        whenNotPaused  // here ! 
        returns (uint256 userId){

& 
AddressDriver.sol#L122
 function setDrips(
        IERC20 erc20,
        DripsReceiver[] calldata currReceivers,
        int128 balanceDelta,
        DripsReceiver[] calldata newReceivers,
        // slither-disable-next-line similar-names
        uint32 maxEndHint1,
        uint32 maxEndHint2,
        address transferTo
    ) public whenNotPaused returns (int128 realBalanceDelta) { // here ! 
        if (balanceDelta > 0) {
            _transferFromCaller(erc20, uint128(balanceDelta));
        }
        realBalanceDelta = dripsHub.setDrips(
            callerUserId(),
            erc20,
            currReceivers,
            balanceDelta,
            newReceivers,
            maxEndHint1,
            maxEndHint2
        );

mess authorization modifire and can be called by any one ! 
----------------------------------------------
DripsHub.sol#134
 function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
        DripsHubStorage storage dripsHubStorage = _dripsHubStorage();
        driverId = dripsHubStorage.nextDriverId++;
        dripsHubStorage.driverAddresses[driverId] = driverAddr;
        emit DriverRegistered(driverId, driverAddr);
    }

#152
    function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
        _assertCallerIsDriver(driverId);
        _dripsHubStorage().driverAddresses[driverId] = newDriverAddr;
        emit DriverAddressUpdated(driverId, msg.sender, newDriverAddr);
    }

mess a modifire and any one can set a DriverAddress and update DriverAddress !
----------------------
AddressDriver.sol#122

   function setDrips(
        IERC20 erc20,
        DripsReceiver[] calldata currReceivers,
        int128 balanceDelta,
        DripsReceiver[] calldata newReceivers,
        // slither-disable-next-line similar-names
        uint32 maxEndHint1,
        uint32 maxEndHint2,
        address transferTo
    ) public whenNotPaused returns (int128 realBalanceDelta) {

any one can set a Drips!

AddressDriver.sol#185

any one can set splits

    function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
        dripsHub.setSplits(callerUserId(), receivers);
    }
