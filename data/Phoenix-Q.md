## Missing address validation

[`AddressDriver.sol#L30`](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30)

**Description**: Constructor parameter in `AddressDriver` is not validated. The immutable variable `trustedForwarder`in `ERC2771Context`. 

```solidity=
constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
    {
        dripsHub = _dripsHub;
        driverId = _driverId;
    }
```

Setting up state storage withou checking the parameter `driverAddr` for `address(0)`

[`DripsHub.sol#L134`](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L134)

```solidity=
    function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
        DripsHubStorage storage dripsHubStorage = _dripsHubStorage();
        driverId = dripsHubStorage.nextDriverId++;
        dripsHubStorage.driverAddresses[driverId] = driverAddr;
        emit DriverRegistered(driverId, driverAddr);
    }
```

## Too centralized admin powers, risk of loosing the protocol due to unadverted admin when changing admin on any `Managed` contract using `changeAdmin`

[`Managed.sol#L84`](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84)

### Suggestion: use two steps changing admin, where the new admin has to confirm (only then the state variable is updated). Another way is to implement admin `ROLES`, where more than one admin can be set 