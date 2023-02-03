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