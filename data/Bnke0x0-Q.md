

### [L01] Missing checks for `address(0x0)` when assigning values to `address` state variables


#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L33 => 'dripsHub = _dripsHub;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L34 => 'driverId = _driverId;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L29 => 'dripsHub = _dripsHub;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L30 => 'driverId = _driverId;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L36 => 'dripsHub = _dripsHub;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L37 => 'driverId = _driverId;'





### [L02] `safeApprove()` is deprecated

#### Impact
[Deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45) in favor of safeIncreaseAllowance() and safeDecreaseAllowance()
#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L174 => 'erc20.safeApprove(address(dripsHub), type(uint256).max);'
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L289 => 'erc20.safeApprove(address(dripsHub), type(uint256).max);'






### [L03] approve should be replaced with safeApprove or safeIncreaseAllowance() / safeDecreaseAllowance()

#### Impact
approve is subject to a known front-running attack. Consider using safeApprove instead:
#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L250 => 'super.approve(to, tokenId);'





### [L04] `_safeMint()` should be used rather than `_mint()` wherever possible

#### Impact
_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function
#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L68 => '_mint(to, tokenId);'





### [L05] Unspecific Compiler Version Pragma

#### Impact
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L2 => 'pragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L2 => 'pragma solidity ^0.8.17;'





### [L06] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE

#### Impact
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30-L35 => '    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
    {
        dripsHub = _dripsHub;
        driverId = _driverId;
    }'




https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28-L32 => '    constructor(DripsHub _dripsHub, uint32 _driverId) {
        dripsHub = _dripsHub;
        driverId = _driverId;
        totalSplitsWeight = _dripsHub.TOTAL_SPLITS_WEIGHT();
    }'



https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32-L38 => 'constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
        ERC721("DripsHub identity", "DHI")
    {
        dripsHub = _dripsHub;
        driverId = _driverId;
    }'


https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94-L96 => '   constructor(bytes32 splitsStorageSlot) {
        _splitsStorageSlot = splitsStorageSlot;
    }'


Events help non-contract tools to track changes, and events prevent users from being surprised by changes

### Recommended Mitigation Steps
Add Event-Emit.

### Non-Critical Issues


### [N01] Adding a return statement when the function defines a named return variable, is redundant



#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L803 => 'return configsLen;'








### [N02] constants should be defined rather than using magic numbers



#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L185 => 'mapping(uint256 => uint32[2 ** 32]) nextSqueezed;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L366 => '_addDeltaRange(state, cycleStart, cycleStart + 1, -int256(amt * _AMT_PER_SEC_MULTIPLIER));'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L419 => 'uint32[2 ** 32] storage nextSqueezed ='






### [N03] Event is missing `indexed` fields

#### Impact
Each event should use three indexed fields if there are three or more fields
#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L25 => 'event PauserGranted(address indexed pauser, address admin);'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L30 => 'event PauserRevoked(address indexed pauser, address admin);'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L33 => 'event Collected(uint256 indexed userId, uint256 indexed assetId, uint128 collected);'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L49 => 'event Collectable(uint256 indexed userId, uint256 indexed assetId, uint128 amt);'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L71 => 'event SplitsReceiverSeen(bytes32 indexed receiversHash, uint256 indexed userId, uint32 weight);'






### [N04] NC-library/interface files should use fixed compiler versions, not floating ones


#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L2 => 'pragma solidity ^0.8.17;'



### [N05] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()


#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L207 => 'return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);'




### [N06] PRAGMA VERSION^0.8.17 VERSION TOO RECENT TO BE TRUSTED.

#### Impact
https://github.com/ethereum/solidity/blob/develop/Changelog.md
0.8.17 (2022-09-08)
0.8.16 (2022-08-08)
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)

Unexpected bugs can be reported in recent versions;
Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs

Use a non-legacy and more battle-tested version
Use 0.8.10

#### Findings:


https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L2 => 'pragma solidity ^0.8.17;'
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L2 => 'pragma solidity ^0.8.17;'


### [N07] SHOWING THE ACTUAL VALUES OF NUMBERS IN NATSPEC COMMENTS MAKES CHECKING AND READING CODE EASIER


#### Findings:



https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L67 => 'uint256 public constant DRIVER_ID_OFFSET = 224;'






