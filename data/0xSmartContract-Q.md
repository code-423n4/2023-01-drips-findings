## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|`collect()` function allows re-entrancy from hookable tokens| 1 |
|[L-02]|Danger "while" loop | 1 |
|[L-03]|Missing Event for  initialize | 5 |
|[L-04]|Lack of control to assign 0 values in the value assignments of critical state variables in the constructor |1 |
|[L-05]|Project Upgrade and Stop Scenario should be| 1 |
|[L-06]|Draft Openzeppelin Dependencies | 1 |
|[L-07]|Loss of precision due to rounding| 1 |
|[L-08]|Some events are missing `msg.sender` parameters| 1 |
|[L-09]|Need Fuzzing test|  |
|[L-10]|Using both `mint` and `safeMint` method at the same time is not the right way for security |  |
|[L-11]|Cross-chain replay attacks are possible with  `callSigned` | 1 |

Total 11 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01] |Implement some type of version counter that will be incremented automatically for contract upgrades|1|
| [N-02] |Insufficient coverage |All Contracts|
| [N-03] |`Function writing` that does not comply with the `Solidity Style Guide`| All Contracts |
| [N-04] |Tokens accidentally sent to the contract cannot be recovered| 1 |
| [N-05] |Assembly Codes Specific – Should Have Comments|12 |
| [N-06] |For functions, follow Solidity standard naming conventions (internal function style rule) | 8 |
| [N-07] |Floating pragma| 8 |
| [N-08] |Use SMTChecker|  |
| [N-09] |Add NatSpec Mapping comment | 16 |
| [N-10] |Remove Unused Codes| 1  |
| [N-11] |Highest risk must be specified in NatSpec comments and documentation|1 |
| [N-12] |Not using the type name in function specified in `returns` causes confusion | 1 |
| [N-13] |Use a single file for all system-wide constants | 16 |

Total 13 issues


### Suggestions
| Number | Suggestion Details |
|:--:|:-------|
| [S-01] |Generate perfect code headers every time |

### [L-01] `collect()` function allows re-entrancy from hookable tokens

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L60-L63


### Vulnerability details
Some token standards, such as ERC777, allow a callback to the source of the funds before the balances are updated in transfer(). This callback could be used to re-enter the protocol while already holding the minted tranche tokens and at a point where the system accounting reflects a receipt of funds that has not yet occurred.

Even if an attacker cannot interact with this function due to any restrictions, they may still interact with the rest of the protocol, so care should be taken with any externall calls that may be affected.

```js
src/AddressDriver.sol:
  59      /// @return amt The collected amount
  60:     function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
  61:         amt = dripsHub.collect(callerUserId(), erc20);
  62:         erc20.safeTransfer(transferTo, amt);
  63:     }
```

### Recommendation;
Add Re-Entrancy Guard to  the function

## [L-02] Danger "while" loop


## Impact
`Drips.sol#L716-L729` has a while loop on this line .
When using `while` loops, the Solidity compiler is aware that a condition needs to be checked first, before executing the code inside the loop.

Firstly, the conditional part is evaluated

a) if the condition evaluates to true , the instructions inside the loop (defined inside the brackets { ... }) get executed.

b) once the code reaches the end of the loop body (= the closing curly brace } , it will jump back up to the conditional part).

c) the code inside the while loop body will keep executing over and over again until the conditional part turns false.

For loops are more associated with iterations. While loop instead are more associated with repetitive tasks. Such tasks could potentially be infinite if the while loop body never affects the condition initially checked.
Therefore, they should be used with care.



## Proof of Concept


At very small intervals, a large number of loops can be accessed by someone attempting a DOS attack.

```js
src/Drips.sol:
  715  
  716:             while (true) {
  717:                 uint256 end = (enoughEnd + notEnoughEnd) / 2;
  718:                 if (end == enoughEnd) {
  719:                     return uint32(end);
  720:                 }
  721:                 if (_isBalanceEnough(balance, configs, configsLen, end)) {
  722:                     enoughEnd = end;
  723:                 } else {
  724:                     notEnoughEnd = end;
  725:                 }
  726:             }
  727:             // slither-disable-end incorrect-equality,timestamp
  728:         }
  729      }

```


## Tools Used
Manual code review

## Recommended Mitigation Steps
Set a reasonable while loop interval

### [L-03] Missing Event for  initialize

**Context:**
```solidity
5 result

src/AddressDriver.sol:
  29      /// @param _driverId The driver ID to use when calling DripsHub.
  30:     constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
  31:         ERC2771Context(forwarder)
  32:     {
  33:         dripsHub = _dripsHub;
  34:         driverId = _driverId;
  35:     }


src/Drips.sol:
  218      /// @param dripsStorageSlot The storage slot to holding a single `DripsStorage` structure.
  219:     constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
  220:         require(cycleSecs > 1, "Cycle length too low");
  221:         _cycleSecs = cycleSecs;
  222:         _dripsStorageSlot = dripsStorageSlot;
  223:     }

src/NFTDriver.sol:
  31      /// @param _driverId The driver ID to use when calling DripsHub.
  32:     constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
  33:         ERC2771Context(forwarder)
  34:         ERC721("DripsHub identity", "DHI")
  35:     {
  36:         dripsHub = _dripsHub;
  37:         driverId = _driverId;
  38:     }


src/Managed.sol:
  72      /// The contract instance can be used only as a call delegation target for a proxy.
  73:     constructor() {
  74:         _managedStorage().isPaused = true;
  75:     }

src/ImmutableSplitsDriver.sol:
  27      /// @param _driverId The driver ID to use when calling DripsHub.
  28:     constructor(DripsHub _dripsHub, uint32 _driverId) {
  29:         dripsHub = _dripsHub;
  30:         driverId = _driverId;
  31:         totalSplitsWeight = _dripsHub.TOTAL_SPLITS_WEIGHT();
  32:     }

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit

### [L-04] Lack of control to assign 0 values in the value assignments of critical state variables in the constructor


```solidity
src/Splits.sol:
  93      /// @param splitsStorageSlot The storage slot to holding a single `SplitsStorage` structure.
  94:     constructor(bytes32 splitsStorageSlot) {
  95:         _splitsStorageSlot = splitsStorageSlot;
  96:     }

```

### [L-05] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

### [L-06] Draft Openzeppelin Dependencies

The `Caller.sol` contract utilised `draft-EIP712.sol`, an OpenZeppelin contract. This contract is still a draft and are not considered ready for mainnet use. OpenZeppelin contracts may be considered draft contracts if they have not received adequate security auditing or are liable to change with future development.


```solidity

src/Caller.sol:
  5: import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";

```

### [L-07] Loss of precision due to rounding


```solidity

src/Drips.sol:
  1119      /// @return cycle The cycle containing the timestamp.
  1120:     function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
  1121:         unchecked {
  1122:             return timestamp / _cycleSecs + 1;
  1123:         }
  1124:     }
```
### [L-08] Some events are missing `msg.sender` parameters


Only amounts are published in emits, whereas msg.sender must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose.
Basically, this event cannot be used

```diff

src/Drips.sol:
  232      /// @return receivedAmt The received amount
  233:     function _receiveDrips(uint256 userId, uint256 assetId, uint32 maxCycles)
  234:         internal
  235:         returns (uint128 receivedAmt)
  236:     {
  237:         uint32 receivableCycles;
  238:         uint32 fromCycle;
  239:         uint32 toCycle;
  240:         int128 finalAmtPerCycle;
  241:         (receivedAmt, receivableCycles, fromCycle, toCycle, finalAmtPerCycle) =
  242:             _receiveDripsResult(userId, assetId, maxCycles);
  243:         if (fromCycle != toCycle) {
  244:             DripsState storage state = _dripsStorage().states[assetId][userId];
  245:             state.nextReceivableCycle = toCycle;
  246:             mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
  247:             for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
  248:                 delete amtDeltas[cycle];
  249:             }
  250:             // The next cycle delta must be relative to the last received cycle, which got zeroed.
  251:             // In other words the next cycle delta must be an absolute value.
  252:             if (finalAmtPerCycle != 0) {
  253:                 amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
  254:             }
- 255:         }  256:         emit ReceivedDrips(userId, assetId, receivedAmt, receivableCycles);
+ 255:         }  256:         emit ReceivedDrips(msg.sender, userId, assetId, receivedAmt, receivableCycles);

  257:     }

```

```diff
src/Drips.sol:
  341      /// @return amt The squeezed amount.
  342:     function _squeezeDrips(
  343:         uint256 userId,
  344:         uint256 assetId,
  345:         uint256 senderId,
  346:         bytes32 historyHash,
  347:         DripsHistory[] memory dripsHistory
  348:     ) internal returns (uint128 amt) {
  349:         uint256 squeezedNum;
  350:         uint256[] memory squeezedRevIdxs;
  351:         bytes32[] memory historyHashes;
  352:         uint256 currCycleConfigs;
  353:         (amt, squeezedNum, squeezedRevIdxs, historyHashes, currCycleConfigs) =
  354:             _squeezeDripsResult(userId, assetId, senderId, historyHash, dripsHistory);
  355:         bytes32[] memory squeezedHistoryHashes = new bytes32[](squeezedNum);
  356:         DripsState storage state = _dripsStorage().states[assetId][userId];
  357:         uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
  358:         for (uint256 i = 0; i < squeezedNum; i++) {
  359:             // `squeezedRevIdxs` are sorted from the newest configuration to the oldest,
  360:             // but we need to consume them from the oldest to the newest.
  361:             uint256 revIdx = squeezedRevIdxs[squeezedNum - i - 1];
  362:             squeezedHistoryHashes[i] = historyHashes[historyHashes.length - revIdx];
  363:             nextSqueezed[currCycleConfigs - revIdx] = _currTimestamp();
  364:         }
  365:         uint32 cycleStart = _currCycleStart();
  366:         _addDeltaRange(state, cycleStart, cycleStart + 1, -int256(amt * _AMT_PER_SEC_MULTIPLIER));
- 367:         emit SqueezedDrips(userId, assetId, senderId, amt, squeezedHistoryHashes);
+ 367:         emit SqueezedDrips(msg.sender, userId, assetId, senderId, amt, squeezedHistoryHashes);
  368:     }
  369 

```

```diff
src/Drips.sol:
  609      /// @return realBalanceDelta The actually applied drips balance change.
  610:     function _setDrips(
                // codes...
- 666:                 emit DripsReceiverSeen(newDripsHash, receiver.userId, receiver.config);
+ 666:                  emit DripsReceiverSeen(msg.sender, newDripsHash, receiver.userId, receiver.config);
  667:             }
  668:         }

```

### Recommended Mitigation Steps
add `msg.sender` parameter in event-emit

### [L-09] Need Fuzzing test

**Context:**

```solidity
6 results - 1 file

src/Drips.sol:
   685      ) private view returns (uint32 maxEnd) {
   686:         unchecked {
   687              (uint256[] memory configs, uint256 configsLen) = _buildConfigs(receivers);

   742      ) private view returns (bool isEnough) {
   743:         unchecked {
   744              uint256 spent = 0;

   773      {
   774:         unchecked {
   775              require(receivers.length <= _MAX_DRIPS_RECEIVERS, "Too many drips receivers");

  1040      ) private {
  1041:         unchecked {
  1042              // In order to set a delta on a specific timestamp it must be introduced in two cycles.

  1099          // per transaction and it needs to be optimized as much as possible.
  1100:         // As of Solidity 0.8.13, rewriting it in unchecked Solidity triples its gas cost.
  1101          uint256 cycleSecs = _cycleSecs;

  1120      function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
  1121:         unchecked {
  1122              return timestamp / _cycleSecs + 1;


```

**Description:**
In total 6 unchecked are used, the functions used are critical. For this reason, there must be fuzzing tests in the tests of the project. Not seen in tests.

**Recommendation:**
Use should fuzzing test like Echidna.

As Alberto Cuesta Canada said:
_Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now._

https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05


### [L-10] Using both `mint` and `safeMint` method at the same time is not the right way for security

For all battle-tested projects, either `mint` or `safeMint` patterns are preferred, this is a design decision; It is evaluated in terms of security concerns and gas optimization, but using both at the same time causes confusion, one of them should be chosen as best practice.

In case of using `mint`, if msg.sender is contract, a faulty contract may be sent without supporting NFT purchase.

```solidity
src/NFTDriver.sol:
  62:     function mint(address to, UserMetadata[] calldata userMetadata)
  63:         public
  64:         whenNotPaused
  65:         returns (uint256 tokenId)
  66:     {
  67:         tokenId = _registerTokenId();
  68:         _mint(to, tokenId);
  69:         if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
  70:     }
  71: 



  79:     function safeMint(address to, UserMetadata[] calldata userMetadata)
  80:         public
  81:         whenNotPaused
  82:         returns (uint256 tokenId)
  83:     {
  84:         tokenId = _registerTokenId();
  85:         _safeMint(to, tokenId);
  86:         if (userMetadata.length > 0) dripsHub.emitUserMetadata(tokenId, userMetadata);
  87:     }
  88 


```

**Recommendation:**
Use only `safeMint`


### [L-11] Cross-chain replay attacks are possible with  `callSigned`

https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L164-L183

If a user does a callSigned() using the wrong network, an attacker can replay the action on the correct chain, and steal the funds a-la the wintermute gnosis safe attack, where the attacker can create the same address that the user tried to, and steal the funds from there
https://mirror.xyz/0xbuidlerdao.eth/lOE5VN-BHI0olGOXe27F0auviIuoSlnou_9t3XRJseY


There is no chain.id in the data

```solidity
src/Caller.sol:
  174          uint256 currNonce = nonce[sender]++;
  175:         bytes32 executeHash = keccak256(
  176:             abi.encode(
  177:                 callSignedTypeHash, sender, to, keccak256(data), msg.value, currNonce, deadline
  178:             )
  179:         );

```



```solidity
function callSigned(
        address sender,
        address to,
        bytes memory data,
        uint256 deadline,
        bytes32 r,
        bytes32 sv
    ) public payable returns (bytes memory returnData) {
        // slither-disable-next-line timestamp
        require(block.timestamp <= deadline, "Execution deadline expired");
        uint256 currNonce = nonce[sender]++;
        bytes32 executeHash = keccak256(
            abi.encode(
                callSignedTypeHash, sender, to, keccak256(data), msg.value, currNonce, deadline
            )
        );
        address signer = ECDSA.recover(_hashTypedDataV4(executeHash), r, sv);
        require(signer == sender, "Invalid signature");
        return _call(sender, to, data, msg.value);
    }


```

Similar issue from Harpieio;
https://github.com/Harpieio/contracts/pull/4/commits/de24a50349ec014163180ba60b5305098f42eb14

 p.s : I couldn't write a POC because I didn't have enough time, so I entred in as a QA

### Recommended Mitigation Steps
Include the chain.id in keccak256


### [N-01] Implement some type of version counter that will be incremented automatically for contract upgrades

As part of the upgradeability of  Proxies , the contract can be upgraded multiple times, where it is a systematic approach to record the version of each upgrade.

```js
src/Managed.sol:
  149  
  150:     /// @notice Authorizes the contract upgrade. See `UUPSUpgradeable` docs for more details.
  151:     function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {
  152:         return;
  153:     }

```
I suggest implementing some kind of version counter that will be incremented automatically when you upgrade the contract.

**Recommendation:**
```js

uint256 public authorizeUpgradeCounter;

function _authorizeUpgrade(address newImplementation) internal override onlyGuardian {
	authorizeUpgradeCounter+=1;
}


    }
```
### [N-02] Insufficient coverage

**Description:**
The test coverage rate of the project is ~80%. Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.


```js
| File                          | % Lines          | % Statements     | % Branches       | % Funcs          |
|-------------------------------|------------------|------------------|------------------|------------------|
| src/AddressDriver.sol         | 100.00% (16/16)  | 100.00% (16/16)  | 100.00% (6/6)    | 87.50% (7/8)     |
| src/Caller.sol                | 100.00% (22/22)  | 100.00% (30/30)  | 100.00% (10/10)  | 100.00% (8/8)    |
| src/Drips.sol                 | 89.01% (243/273) | 89.27% (283/317) | 66.98% (71/106)  | 83.33% (30/36)   |
| src/DripsHub.sol              | 100.00% (58/58)  | 100.00% (63/63)  | 100.00% (14/14)  | 100.00% (31/31)  |
| src/ImmutableSplitsDriver.sol | 100.00% (10/10)  | 100.00% (13/13)  | 75.00% (3/4)     | 100.00% (2/2)    |
| src/Managed.sol               | 94.12% (16/17)   | 94.12% (16/17)   | 100.00% (4/4)    | 100.00% (12/12)  |
| src/NFTDriver.sol             | 87.10% (27/31)   | 87.88% (29/33)   | 80.00% (8/10)    | 94.44% (17/18)   |
| src/Splits.sol                | 100.00% (64/64)  | 100.00% (71/71)  | 68.18% (15/22)   | 100.00% (13/13)  |
| Total                         | 81.58% (465/570) | 82.04% (530/646) | 65.20% (133/204) | 84.25% (123/146) |

```

### [N-03] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-04] Tokens accidentally sent to the contract cannot be recovered

Context;
AddressDriver.sol

It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```

### [N-05] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does


This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity

12 results - 4 files

src/Drips.sol:
   820          uint256 val;
   821:         // slither-disable-next-line assembly
   822:         assembly ("memory-safe") {
   823              val := mload(add(32, add(configs, shl(5, idx))))

  1101          uint256 cycleSecs = _cycleSecs;
  1102:         // slither-disable-next-line assembly
  1103:         assembly {
  1104              let endedCycles := sub(div(end, cycleSecs), div(start, cycleSecs))

  1143          bytes32 slot = _dripsStorageSlot;
  1144:         // slither-disable-next-line assembly
  1145:         assembly {
  1146              dripsStorage.slot := slot

src/DripsHub.sol:
  622          bytes32 slot = _dripsHubStorageSlot;
  623:         // slither-disable-next-line assembly
  624:         assembly {
  625              storageRef.slot := slot

src/Managed.sol:
  143          bytes32 slot = _managedStorageSlot;
  144:         // slither-disable-next-line assembly
  145:         assembly {
  146              storageRef.slot := slot

src/Splits.sol:
  284          bytes32 slot = _splitsStorageSlot;
  285:         // slither-disable-next-line assembly
  286:         assembly {
  287              splitsStorage.slot := slot
```

### [N-06] For functions, follow Solidity standard naming conventions (internal function style rule)

**Context:**
```solidity

8 results
8 results

src/AddressDriver.sol:
  46:     function callerUserId() internal view returns (uint256 userId) {

src/Caller.sol:
  84:     bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));

src/Drips.sol:
  60:     function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_) internal
  72:     function dripId(DripsConfig config) internal pure returns (uint32) {
  77:     function amtPerSec(DripsConfig config) internal pure returns (uint160) {
  82:     function start(DripsConfig config) internal pure returns (uint32) {
  87:     function duration(DripsConfig config) internal pure returns (uint32) {
  93:     function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {

```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)


### [N-07] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 
```solidity
8 results - 8 files

src/AddressDriver.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
  3  

src/Caller.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
  3  

src/Drips.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
  3  

src/DripsHub.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
  3  

src/ImmutableSplitsDriver.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
  3  

src/Managed.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
  3  

src/NFTDriver.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
  3  

src/Splits.sol:
  1  // SPDX-License-Identifier: GPL-3.0-only
  2: pragma solidity ^0.8.17;
```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### [N-08] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

### [N-09]  Add NatSpec Mapping comment

**Description:**
Add NatSpec comments describing mapping keys and values


**Context:**

```solidity
16 results - 4 files

src/Caller.sol:
  88:     mapping(address => EnumerableSet.AddressSet) internal _authorized;
  90:     mapping(address => uint256) public nonce;
  91  

src/Drips.sol:
   175:   mapping(uint256 => mapping(uint256 => DripsState)) states;
   184:   mapping(uint256 => uint32[2 ** 32]) nextSqueezed;
   202:   mapping(uint32 => AmtDelta) amtDeltas;
   245:   mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
   872:   mapping(uint256 => DripsState) storage states,
  1026:   mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
  1036:   mapping(uint32 => AmtDelta) storage amtDeltas,

src/DripsHub.sol:
   99:    mapping(uint32 => address) driverAddresses;
  101:    mapping(IERC20 => uint256) totalBalances;
  630:    mapping(IERC20 => uint256) storage totalBalances = _dripsHubStorage().totalBalances;

src/Splits.sol:
   76:    mapping(uint256 => SplitsState) splitsStates;
   83:    mapping(uint256 => SplitsBalance) balances;
  148:    mapping(uint256 => SplitsState) storage splitsStates = _splitsStorage().splitsStates;

```

**Recommendation:**

```solidity

 /// @dev    address(user) -> address(ERC0 Contract Address) -> uint256(allowance amount from user)
    mapping(address => mapping(address => uint256)) public allowance;

```


### [N-10] Remove Unused Codes

`return` keyword is not necessary

```solidity
src/DripsHub.sol:
  108      /// Must be higher than 1
  109:     constructor(uint32 cycleSecs_)
  110:         Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
  111:         Splits(_erc1967Slot("eip1967.splits.storage"))
  112:     {
  113:         return;
  114:     }

```

### [N-11] Highest risk must be specified in NatSpec comments and documentation

```solidity

src/Managed.sol:
  149  
  150:     /// @notice Authorizes the contract upgrade. See `UUPSUpgradeable` docs for more details.
  151:     function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {
  152:         return;
  153:     }
```

**Description:**
Although the UUPS model has many advantages and the proposed proxy model is currently the UUPS model, there are some caveats we should be aware of before applying it to a real-world project.

One of the main attention: Since upgrades are done through the implementation contract with the help of the 
``` _authorizeUpgrade ``` method, there is a high risk of new implementations such as excluding the ``` _authorizeUpgrade ``` method, which can permanently kill the smart contract's ability to upgrade. Also, this pattern is somewhat complicated to implement compared to other proxy patterns.

**Recommendation:**
Therefore, this highest risk must be found in NatSpec comments and documents.

### [N-12] Not using the type name in function specified in `returns` causes confusion

`Drips._squeezed()` function has `uint128` type and `squeezedAmt`name returns argument , but this name never use in function , thats causes confusion 

```diff

src/Drips.sol:
  469      /// @return squeezedAmt The squeezed amount.
  470:     function _squeezedAmt(
  471:         uint256 userId,
  472:         DripsHistory memory dripsHistory,
  473:         uint32 squeezeStartCap,
  474:         uint32 squeezeEndCap
- 475:     ) private view returns (uint128 squeezedAmt) {
+ 475:     ) private view returns (uint128) {

  476:         DripsReceiver[] memory receivers = dripsHistory.receivers;
  477:         // Binary search for the `idx` of the first occurrence of `userId`
  478:         uint256 idx = 0;
  479:         for (uint256 idxCap = receivers.length; idx < idxCap;) {
  480:             uint256 idxMid = (idx + idxCap) / 2;
  481:             if (receivers[idxMid].userId < userId) {
  482:                 idx = idxMid + 1;
  483:             } else {
  484:                 idxCap = idxMid;
  485:             }
  486:         }
  487:         uint32 updateTime = dripsHistory.updateTime;
  488:         uint32 maxEnd = dripsHistory.maxEnd;
  489:         uint256 amt = 0;
  490:         for (; idx < receivers.length; idx++) {
  491:             DripsReceiver memory receiver = receivers[idx];
  492:             if (receiver.userId != userId) break;
  493:             (uint32 start, uint32 end) =
  494:                 _dripsRange(receiver, updateTime, maxEnd, squeezeStartCap, squeezeEndCap);
  495:             amt += _drippedAmt(receiver.config.amtPerSec(), start, end);
  496:         }
  497:         return uint128(amt);
  498:     }

```



**Recommendation:**
Consider adopting a consistent approach to return values throughout the codebase by removing all named return variables, explicitly declaring them as local variables, and adding the necessary return statements where appropriate. This would improve both the explicitness and readability of the code, and it may also help reduce regressions during future code refactors.


### [N-13] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

  This will help with readability and easier maintenance for future changes. 

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution

```solidity

16 results - 4 files

src/Caller.sol:
  81  contract Caller is EIP712("Caller", "1"), ERC2771Context(address(this)) {
  82:     string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("
  83          "address sender,address to,bytes data,uint256 value,uint256 nonce,uint256 deadline)";

src/Drips.sol:
  105      /// Limits cost of changes in drips configuration.
  106:     uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;
  107      /// @notice The additional decimals for all amtPerSec values.
  108:     uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
  109      /// @notice The multiplier for all amtPerSec values. It's `10 ** _AMT_PER_SEC_EXTRA_DECIMALS`.
  110:     uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
  111      /// @notice The total amount the contract can keep track of each asset.
  112:     uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
  113      /// @notice On every timestamp `T`, which is a multiple of `cycleSecs`, the receivers

src/DripsHub.sol:
  34  /// so recently dripped funds may not be receivable immediately.
  35: /// `cycleSecs` is a constant configured when the drips hub is deployed.
  36  /// The receiver balance is independent from the drips balance,

  55      /// Limits cost of changes in drips configuration.
  56:     uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;
  57      /// @notice The additional decimals for all amtPerSec values.
  58:     uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
  59      /// @notice The multiplier for all amtPerSec values.
  60:     uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
  61      /// @notice Maximum number of splits receivers of a single user. Limits the cost of splitting.
  62:     uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
  63      /// @notice The total splits weight of a user
  64:     uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
  65      /// @notice The offset of the controlling driver ID in the user ID.
  66      /// In other words the controlling driver ID is the highest 32 bits of the user ID.
  67:     uint256 public constant DRIVER_ID_OFFSET = 224;
  68      /// @notice The total amount the contract can store of each token.
  69      /// It's the minimum of _MAX_TOTAL_DRIPS_BALANCE and _MAX_TOTAL_SPLITS_BALANCE.
  70:     uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;
  71      /// @notice The ERC-1967 storage slot holding a single `DripsHubStorage` structure.

src/Splits.sol:
  19      /// @notice Maximum number of splits receivers of a single user. Limits the cost of splitting.
  20:     uint256 internal constant _MAX_SPLITS_RECEIVERS = 200;
  21      /// @notice The total splits weight of a user
  22:     uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
  23      /// @notice The total amount the contract can keep track of each asset.
  24      // slither-disable-next-line unused-state
  25:     uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;
  26      /// @notice The storage slot holding a single `SplitsStorage` structure.

```

### [S-01] Generate perfect code headers every time

**Description:**
I recommend using header for Solidity code layout and readability

https://github.com/transmissions11/headers

```js
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```
