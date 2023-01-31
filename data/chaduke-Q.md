QA3. It is advised to follow the common practice: to choose a slot that has no KNOWN preimage. It is much easier to find a second preimage if we already know the first preimage. 
 
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L72

Mitigation: 
Add -1 to avoid known preimage. 

```
bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage") - 1;
```

QA4. If a driver set the wrong (possibly inaccessible) ``newDriverAddr``, then he can will lose his driver's privilege forever. 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152-L156
```
 function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
        _assertCallerIsDriver(driverId);
        _dripsHubStorage().driverAddresses[driverId] = newDriverAddr;
        emit DriverAddressUpdated(driverId, msg.sender, newDriverAddr);
    }
```

Mitigation: This is similar to the change of the owner of a contract. We need to  follow a two-step procedure: 1) first set the new pending address; 2) use the new pending address to accept the change to the final new driver address. 


QA5. It is important to emit events when critical state variables are changed by some functions. 

a) https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L629-L637



QA6. [_addDeltaRange()](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1020-L1030) fails to check the input range is valid. 
One caller [_updateReceiverStates()](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L872-L965) forgot to check this and possibly pass invalid ranges to this function, causing a chaos to the system.

1) The following code does not check that ``start <= end`` and even when ``start > end``, it still performs the update. 
```javascript
function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)
        private
    {
        // slither-disable-next-line incorrect-equality,timestamp
        if (start == end) {
            return;
        }
        mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
        _addDelta(amtDeltas, start, amtPerSec);
        _addDelta(amtDeltas, end, -amtPerSec);
    }
```

2) ``_updateReceiverStates()`` calls ``_addDeltaRange()`` in the [following code](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L919-L920):
```javascript
                     _addDeltaRange(state, currStart, newStart, -amtPerSec);
                    _addDeltaRange(state, currEnd, newEnd, amtPerSec);
```
3) It is possible that (currStart > newStart) and (currEnd > newEnd). In this case, the ``_updateReceiverStates()`` will make the wrong updates. The above code has its own other problem (wrong range applied), but here we illustates another problem: when both the caller and callee does not perform a valid range check, there is a bad consequence.

4) This scenario shows the importance of doing a valid range check for ``_addDeltaRange()``.

Mitigation:
We add a valid range check below:
```diff
function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)
        private
    {
+       if(start > end) revert InvalidRange();

        // slither-disable-next-line incorrect-equality,timestamp
        if (start == end) {
            return;
        }
        mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
        _addDelta(amtDeltas, start, amtPerSec);
        _addDelta(amtDeltas, end, -amtPerSec);
    }
```

QA7. False emit is possible here. 

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L215
```
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
We need to check whether it is a new receiver list. The emit statement needs to be in the if block.
```diff
function _setSplits(uint256 userId, SplitsReceiver[] memory receivers) internal {
        SplitsState storage state = _splitsStorage().splitsStates[userId];
        bytes32 newSplitsHash = _hashSplits(receivers);
-        emit SplitsSet(userId, newSplitsHash);
        if (newSplitsHash != state.splitsHash) {
            _assertSplitsValid(receivers, newSplitsHash);
            state.splitsHash = newSplitsHash;
+          emit SplitsSet(userId, newSplitsHash);
        }
    }
```
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152-L155

We need to check whether the driver's new address is the same as the old one. 

QA7. ``DripsHub`` and ``ImmutableSplitDriver`` have a serious problem for the implementation of UUPSUpgradeable:
1) There is no ``Initialize()`` function in the implementation of ``Driphub and ``Managed`` and ``ImmutableSplitDriver``. The constructors do not initialize the context for the proxy. ``Initialize()`` does.  For example, there is no way to initialize ``_cycleSecs``, ``_dripsStorageSlot``, ``_splitsStorageSlot`` for ``DripsHub`` from the proxy. There is no way to initialize ``dripsHub``,   ``driverId``,  ``totalSplitsWeight`` for ``ImmutableSplitDriver`` from the proxy either.
 
```
 constructor(uint32 cycleSecs_)
        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
        Splits(_erc1967Slot("eip1967.splits.storage"))
    {
        return;
    }

 constructor() {
        _managedStorage().isPaused = true;
    }
```

2) There are no ``_disableInitializers();`` called in the constructor, which is necessary to prevent initialization of the implementation contract itself, as extra protection to prevent an attacker from initializing it.
```javascript
// @custom:oz-upgrades-unsafe-allow constructor
constructor() {
    _disableInitializers();
}
```

Mitigation:
1) implement the initialize function  - do not rely on constructors to perform initialization, which will not work for proxies.  


2)  include ``_disableInitializers();`` in the constructor.

QA8. It is advised that all constants should be defined in one file:
```javascript
  uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;
    /// @notice The additional decimals for all amtPerSec values.
    uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
    /// @notice The multiplier for all amtPerSec values.
    uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
    /// @notice Maximum number of splits receivers of a single user. Limits the cost of splitting.
    uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
    /// @notice The total splits weight of a user
    uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
    /// @notice The offset of the controlling driver ID in the user ID.
    /// In other words the controlling driver ID is the highest 32 bits of the user ID.
    uint256 public constant DRIVER_ID_OFFSET = 224;
    /// @notice The total amount the contract can store of each token.
    /// It's the minimum of _MAX_TOTAL_DRIPS_BALANCE and _MAX_TOTAL_SPLITS_BALANCE.
    uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;
    /// @notice The ERC-1967 storage slot holding a single `DripsHubStorage` structure.
    bytes32 private immutable _dripsHubStorageSlot = _erc1967Slot("eip1967.dripsHub.storage");
uint256 internal constant _MAX_SPLITS_RECEIVERS = 200;
    /// @notice The total splits weight of a user
    uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
    /// @notice The total amount the contract can keep track of each asset.
    // slither-disable-next-line unused-state
    uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;
    /// @notice The storage slot holding a single `SplitsStorage` structure.
    bytes32 private immutable _splitsStorageSlot;
    uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;
    /// @notice The additional decimals for all amtPerSec values.
    uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
    /// @notice The multiplier for all amtPerSec values. It's `10 ** _AMT_PER_SEC_EXTRA_DECIMALS`.
    uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
    /// @notice The total amount the contract can keep track of each asset.
    uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
```

QA9. Use a two-step ownership transfer procedure ``safeTransferOwnership`` instead of a one-step admin changing function. Changing admin in one step is risky: if the new address is a wrong input by mistake, we will lose all the privileges of the owner. 

Recommendation:  Use OpenZeppelin's Ownable2Step. https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol

QA10. Use a two-step registration procedure to register a new address to avoid spamming and input mistake: 
1) First step is to propose the new address; 2) use the new address to accept the registration. 
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L134-L139

QA11. Unless a driver sets the new address to the address of contract ``ImmutableSplitsDriver``, otherwise the call of ``ImmutableSplitsDriver.createSplits()`` will always fail. This needs to be well documented. 

1) When a user calls the [ImmutableSplitsDriver.createSplits()](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L53-L68) function, the function calls ``dripsHub.setSplits``.

3) ``dripsHub.setSplits()`` has a modifier ``onlyDriver()`` which checks if the caller is the driver's registered address:
```javascript
 modifier onlyDriver(uint256 userId) {
        uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
        _assertCallerIsDriver(driverId);
        _;
    }

function _assertCallerIsDriver(uint32 driverId) internal view {
        require(driverAddress(driverId) == msg.sender, "Callable only by the driver");
    }
```

4) The driver much register the ``ImmutableSplitsDriver`` as the address, otherwise, the check will fail. As a result both ``dripsHub.setSplits()`` and ``ImmutableSplitsDriver.createSplits()`` will always fail due to lack of address registration.

Mitigation: add NatSpec to ``createSplits`` that a driver must register the address of the ``ImmutableSplitsDriver`` contract  first before calling this function. 


