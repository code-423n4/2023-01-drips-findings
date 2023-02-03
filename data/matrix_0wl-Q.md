# Summary

## Low Risk and Non-Critical Issues

## Non-Critical Issues

|      | Issue                                                      |
| ---- | :--------------------------------------------------------- |
| NC-1 | INSUFFICIENT COVERAGE                                      |
| NC-2 | ADD A TIMELOCK TO CRITICAL FUNCTIONS                       |
| NC-3 | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)       |
| NC-4 | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS          |
| NC-5 | Omissions in events                                        |
| NC-6 | PRAGMA VERSION^0.8.17 VERSION TOO RECENT TO BE TRUSTED     |
| NC-7 | For functions, follow solidity standard naming conventions |
| NC-8 | Lines are too long                                         |
| NC-9 | NOT VERIFIED INPUT                                         |

### [NC-1] INSUFFICIENT COVERAGE

Testing all functions is best practice in terms of security criteria.

#### **Proof Of Concept**

| File                          | % Lines          | % Statements     | % Branches       | % Funcs          |
| ----------------------------- | ---------------- | ---------------- | ---------------- | ---------------- |
| src//Splits.sol               | 0.00% (0/64)     | 0.00% (0/71)     | 0.00% (0/22)     | 0.00% (0/13)     |
| src/AddressDriver.sol         | 100.00% (16/16)  | 100.00% (16/16)  | 100.00% (6/6)    | 87.50% (7/8)     |
| src/Caller.sol                | 100.00% (22/22)  | 100.00% (30/30)  | 100.00% (10/10)  | 100.00% (8/8)    |
| src/Drips.sol                 | 89.01% (243/273) | 89.27% (283/317) | 66.98% (71/106)  | 83.33% (30/36)   |
| src/DripsHub.sol              | 100.00% (58/58)  | 100.00% (63/63)  | 100.00% (14/14)  | 100.00% (31/31)  |
| src/ImmutableSplitsDriver.sol | 100.00% (10/10)  | 100.00% (13/13)  | 75.00% (3/4)     | 100.00% (2/2)    |
| src/Managed.sol               | 94.12% (16/17)   | 94.12% (16/17)   | 100.00% (4/4)    | 100.00% (12/12)  |
| src/NFTDriver.sol             | 87.10% (27/31)   | 87.88% (29/33)   | 80.00% (8/10)    | 94.44% (17/18)   |
| src/Splits.sol                | 100.00% (64/64)  | 100.00% (71/71)  | 68.18% (15/22)   | 100.00% (13/13)  |
| test/Caller.t.sol             | 100.00% (8/8)    | 100.00% (8/8)    | 100.00% (2/2)    | 100.00% (2/2)    |
| test/Drips.t.sol              | 0.00% (0/6)      | 0.00% (0/6)      | 0.00% (0/4)      | 0.00% (0/2)      |
| test/Managed.t.sol            | 100.00% (1/1)    | 100.00% (1/1)    | 100.00% (0/0)    | 100.00% (1/1)    |
| Total                         | 81.58% (465/570) | 82.04% (530/646) | 65.20% (133/204) | 84.25% (123/146) |

Due to its capacity, test coverage is expected to be 100%.

### [NC-2] ADD A TIMELOCK TO CRITICAL FUNCTIONS

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

122:     function setDrips(

158:     function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {

```

```solidity
File: DripsHub.sol

510:     function setDrips(

576:     function setSplits(uint256 userId, SplitsReceiver[] memory receivers)

```

```solidity
File: NFTDriver.sol

186:     function setDrips(

220:     function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)

272:     function setApprovalForAll(address operator, bool approved) public override whenNotPaused {

```

### [NC-3] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled.

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked(,)`.

#### **Proof Of Concept**

```solidity
File: Caller.sol

207:         return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);

```

### [NC-4] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

#### Description:

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html.

NatSpec is missing for the following functions, constructor and modifier:

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

170:     function _transferFromCaller(IERC20 erc20, uint128 amt) internal {

```

```solidity
File: Splits.sol

98:     function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal {

```

```solidity
File: NFTDriver.sol

40:     modifier onlyHolder(uint256 tokenId) {

285:     function _transferFromCaller(IERC20 erc20, uint128 amt) internal {

294:     function _msgSender() internal view override(Context, ERC2771Context) returns (address) {

300:     function _msgData() internal view override(Context, ERC2771Context) returns (bytes calldata) {

```

```solidity
File: DripsHub.sol

124:     function _assertCallerIsDriver(uint32 driverId) internal view {

629:     function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {

635:     function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {

```

```solidity
File: Caller.sol

202:     function _call(address sender, address to, bytes memory data, uint256 value)

```

### [NC-5] Omissions in events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

The events should include the new value and old value where possible.

#### **Proof Of Concept**

Events with no old value:

```solidity
File: Managed.sol

124:         emit Paused(msg.sender);

130:         emit Unpaused(msg.sender);

```

### [NC-6] PRAGMA VERSION^0.8.17 VERSION TOO RECENT TO BE TRUSTED

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

2: pragma solidity ^0.8.17;

```

```solidity
File: Caller.sol

2: pragma solidity ^0.8.17;

```

```solidity
File: Drips.sol

2: pragma solidity ^0.8.17;

```

```solidity
File: DripsHub.sol

2: pragma solidity ^0.8.17;

```

```solidity
File: ImmutableSplitsDriver.sol

2: pragma solidity ^0.8.17;

```

```solidity
File: Managed.sol

2: pragma solidity ^0.8.17;

```

```solidity
File: NFTDriver.sol

2: pragma solidity ^0.8.17;

```

```solidity
File: Splits.sol

2: pragma solidity ^0.8.17;

```

### [NC-7] For functions, follow solidity standard naming conventions

Solidity standard naming convention for internal and private functions: the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

46:     function callerUserId() internal view returns (uint256 userId) {

```

```solidity
File: Drips.sol

73:     function dripId(DripsConfig config) internal pure returns (uint32) {

78:     function amtPerSec(DripsConfig config) internal pure returns (uint160) {

83:     function start(DripsConfig config) internal pure returns (uint32) {

88:     function duration(DripsConfig config) internal pure returns (uint32) {

94:     function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {

```

#### **Recommendation**

Use solidity standard naming convention for internal and private functions

### [NC-8] Lines are too long

#### **Context:**

All Contracts apart from quest-protocol/contracts/interfaces/IQuest.sol

#### **Description:**

Usually lines in source code are limited to 80 characters.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Recommendation**

The lines should be split when they reach that length.

### [NC-9] NOT VERIFIED INPUT

External / public functions parameters should be validated to make sure the address is not 0.

Otherwise if not given the right input it can mistakenly lead to loss of user funds.

#### **Proof Of Concept**

```solidity
File: AddressDriver.sol

40:     function calcUserId(address userAddr) public view returns (uint256 userId) {

60:     function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {

```

```solidity
File: Caller.sol

106:     function authorize(address user) public {

114:     function unauthorize(address user) public {

124:     function isAuthorized(address sender, address user) public view returns (bool authorized) {

132:     function allAuthorized(address sender) public view returns (address[] memory authorized) {

```

```solidity
File: DripsHub.sol

134:     function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {

145:     function driverAddress(uint32 driverId) public view returns (address driverAddr) {

152:     function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {

```

```solidity
File: Managed.sol

84:     function changeAdmin(address newAdmin) public onlyAdmin {

90:     function grantPauser(address pauser) public onlyAdmin {

97:     function revokePauser(address pauser) public onlyAdmin {

105:     function isPauser(address pauser) public view returns (bool isAddrPauser) {

```

```solidity
File: NFTDriver.sol

249:     function approve(address to, uint256 tokenId) public override whenNotPaused {

272:     function setApprovalForAll(address operator, bool approved) public override whenNotPaused {

```

## Low Issues

|     | Issue                                          |
| --- | :--------------------------------------------- |
| L-1 | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE |
| L-2 | DRAFT OPENZEPPELIN DEPENDENCIES                |
| L-3 | LOSS OF PRECISION DUE TO ROUNDING              |
| L-4 | Block values as a proxy for time               |
| L-5 | Unspecific compiler version pragma             |

### [L-1] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

The critical procedures should be two step process.

Changing critical addresses in contracts should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1488) and [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2369))

#### **Proof Of Concept**

```solidity
File: DripsHub.sol

642:     function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {

```

```solidity
File: NFTDriver.sol

272:     function setApprovalForAll(address operator, bool approved) public override whenNotPaused {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-2] DRAFT OPENZEPPELIN DEPENDENCIES

Caller.sol contract utilised draft-EIP712.sol an OpenZeppelin contracts. This contract is still a draft and is not considered ready for mainnet use.

OpenZeppelin contracts may be considered draft contracts if they have not received adequate security auditing or are liable to change with future development.

#### **Proof Of Concept**

```solidity
File: Caller.sol

5: import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";

```

### [L-3] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: Drips.sol

1045:             int256 fullCycle = (int256(uint256(_cycleSecs)) * amtPerSec) / amtPerSecMultiplier;

1047:             int256 nextCycle = (int256(timestamp % _cycleSecs) * amtPerSec) / amtPerSecMultiplier;

```

```solidity
File: Splits.sol

130:         splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);

161:                 uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);

```

### [L-4] Block values as a proxy for time

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as `block.timestamp`, and `block.number` can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of `block.timestamp`, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

https://swcregistry.io/docs/SWC-116

#### **Proof Of Concept**

```solidity
File: Caller.sol

173:         require(block.timestamp <= deadline, "Execution deadline expired");

```

```solidity
File: Drips.sol

1129:         return uint32(block.timestamp);

```

### [L-5] Unspecific compiler version pragma

#### **Context:**

All Contracts
