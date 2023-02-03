### [L-01] Lack of address(0) validation
- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152-L156

```solidity
    function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
        _assertCallerIsDriver(driverId);
        _dripsHubStorage().driverAddresses[driverId] = newDriverAddr;
        emit DriverAddressUpdated(driverId, msg.sender, newDriverAddr);
    }
```

If the original driver sets `newDriverAddr` to address(0) by fault, all funds of this driver will be uncollectable.


### [L-02] The protocol will stop working in 2106

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1128-L1130

```solidity
    function _currTimestamp() private view returns (uint32 timestamp) {
        return uint32(block.timestamp); 
    }
```

Currently, it uses `uint32` for timestamp everywhere, so the protocol won't work properly when `block.timestamp > type(uint32).max`.

For example, the below part won't work at all if `toCycle < toCycle`.

```solidity
File: 2023-01-drips\src\Drips.sol
314:     function _receivableDripsCyclesRange(uint256 userId, uint256 assetId)
315:         private
316:         view
317:         returns (uint32 fromCycle, uint32 toCycle)
318:     {
319:         fromCycle = _dripsStorage().states[assetId][userId].nextReceivableCycle;
320:         toCycle = _cycleOf(_currTimestamp());
321:         // slither-disable-next-line timestamp
322:         if (fromCycle == 0 || toCycle < fromCycle) { //@audit don't work when toCycle is downcasted
323:             toCycle = fromCycle;
324:         }
325:     }
```

### [L-03] lt() function doesn't work as expected.

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L92-L96

```solidity
    /// @notice Compares two `DripsConfig`s.
    /// First compares their `amtPerSec`s, then their `start`s and then their `duration`s.
    function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {
        return DripsConfig.unwrap(config) < DripsConfig.unwrap(otherConfig);
    }
```

As we can read from the comment, it should compare `amtPerSec` first, then `start` and `duration`.

But the high 32 bits are for `dripId` which isn't used now and users can set any value for `dripId`.

So there is no guarantee that `dripId`s will be the same for all users and `lt()` will compare `dripId` first.

We don't use `dripId` for now, I think we can validate `dripId = 0` for all users.


### [L-04] Not implemented like comment

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L585

```solidity
    /// @param newReceivers The list of the drips receivers of the user to be set.
    /// Must be sorted, deduplicated and without 0 amtPerSecs.
```

It says all users should be deduplicated but 2 users that have the same `amtPerSec, start, duration` can pass `lt()` function using different `dripId`.

```solidity
    function _isOrdered(DripsReceiver memory prev, DripsReceiver memory next)
        private
        pure
        returns (bool)
    {
        if (prev.userId != next.userId) {
            return prev.userId < next.userId;
        }
        return prev.config.lt(next.config);
    }
```

### [L-05] Needless modifier

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L136

```solidity
    function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)
        public
        whenNotPaused
        onlyHolder(tokenId)
    {
        _transferFromCaller(erc20, amt);
        dripsHub.give(tokenId, receiver, erc20, amt);
    }
```

This function is used to give funds to the receiver and the funds are transferred from `msg.sender` directly.

So it doesn't change anything of `tokenId` and `tokenId` is only used for an event.

I think it's normal for anyone can give funds to another even if he isn't the owner of any `tokenId`.