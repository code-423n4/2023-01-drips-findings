## [L-01] DRAFT OPENZEPPELIN DEPENDENCIES
The  `Caller.sol`  contract utilised  `draft-EIP712.sol` , an OpenZeppelin contract. This contracts is still a draft and are not considered ready for mainnet use.

OpenZeppelin contracts may be considered draft contracts if they have not received adequate security auditing or are liable to change with future development.

```
src/Caller.sol
5:import {ECDSA, EIP712} from  "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";
```

## [N-01]  _LOCK PRAGMAS_  TO SPECIFIC COMPILER VERSION
**Description:**  
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.  
[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

**Recommendation:**  
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.  
[solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)
```
8 results-8 files
src/Drips.sol:
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma solidity ^0.8.17;

src/DripsHub.sol
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma solidity ^0.8.17;

src/Caller.sol
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma  solidity  ^0.8.17;

src/AddressDriver.sol
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma solidity ^0.8.17;

src/ImmutableSplitsDriver.sol
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma  solidity  ^0.8.17;

src/Managed.sol
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma solidity ^0.8.17;

src/NFTDriver.sol
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma  solidity  ^0.8.17;

src/Splits.sol
1// SPDX-License-Identifier: GPL-3.0-only
2:pragma  solidity  ^0.8.17;

```
## [N-02] USE OF  `BYTES.CONCAT()`  INSTEAD OF  `ABI.ENCODEPACKED()`

```
1 result-1 file
src/Caller.sol:
207:return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);

```
Rather than using  `abi.encodePacked`  for appending bytes, since version 0.8.4,  `bytes.concat()`  is enabled.
Since version 0.8.4 for appending bytes,  `bytes.concat()`  can be used instead of  `abi.encodePacked(,)`

## [N-03] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values):

This will help with readability and easier maintenance for future changes.

**constants.sol**

Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution
```
15 results - 3 files
src/Drips.sol
106:uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;

108:uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;

110:uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;

112:uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);


/src/DripsHub.sol
56:uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;

58:uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;

60:uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;

62:uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;

64:uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;

67:uint256 public constant DRIVER_ID_OFFSET = 224;

70:uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;


src/Splits.sol
20:uint256 internal constant _MAX_SPLITS_RECEIVERS = 200;

22:uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;

25:uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;

27:bytes32 private immutable _splitsStorageSlot;

```
## [N-04] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE
**Description:**  
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

**Recommendation:**  
Add Event-Emit
Context:
[AddressDriver.sol#L30-35](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30-35)
```
constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
	ERC2771Context(forwarder)
{
	dripsHub = _dripsHub;
	driverId = _driverId;
}
```
[Drips.sol#L219-L223](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219-L223)
```
constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
	require(cycleSecs > 1, "Cycle length too low");
	_cycleSecs = cycleSecs;
	_dripsStorageSlot = dripsStorageSlot;
}
```
[ImmutableSplitsDriver.sol#L28-L32](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28-L32)
```
constructor(DripsHub _dripsHub, uint32 _driverId) {
	dripsHub = _dripsHub;
	driverId = _driverId;
	totalSplitsWeight = _dripsHub.TOTAL_SPLITS_WEIGHT();
}
```
[NFTDriver.sol#L32-L38](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32-L38)
```
constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
	ERC2771Context(forwarder)
	ERC721("DripsHub identity", "DHI")
{
	dripsHub = _dripsHub;
	driverId = _driverId;
}

```
[Splits.sol#L94-L96](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94-L96)
```
constructor(bytes32 splitsStorageSlot) {
	_splitsStorageSlot = splitsStorageSlot;
}
```
## [S-01] GENERATE PERFECT CODE HEADERS EVERY TIME
I recommend using header for Solidity code layout and readability:

[https://github.com/transmissions11/headers](https://github.com/transmissions11/headers)

```
/*//////////////////////////////////////////////////////////////  
                         TESTING 123
//////////////////////////////////////////////////////////////*/
```