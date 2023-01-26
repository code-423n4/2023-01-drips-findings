# QA REPORT

---

### Summary of low risk issues


| Number     | Issue details                                                         | Instances |
| ------------ | ----------------------------------------------------------------------- | :---------: |
| [L-1](#L1) | Use of`block.timestamp`                                               |     2     |
| [L-2](#L3) | `_safeMint()` should be used rather than `_mint()` wherever possible. |     1     |
| [L-3](#L4) | Lock pragmas to specific compiler version.                            |     8     |

*Total: 3 issues.*

### Summary of non-critical issues


| Number       | Issue details                                                   | Instances |
| -------------- | ----------------------------------------------------------------- | :---------: |
| [NC-1](#NC1) | Constants should be defined rather than using magic numbers     |     9     |

*Total: 1 issue.*

---

### Low Risk Issues

### <a id=L1>[L-1]</a> Use of `block.timestamp`

##### Description

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the *[Entropy Illusion](https://hacken.io/discover/most-common-smart-contract-vulnerabilities/#Entropy_Illusion)* for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

##### Recommendation

Use oracle instead of `block.timestamp`

##### *Instances (2):*

File: [2023-01-drips/src/Caller.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L173 )

```solidity
173: require(block.timestamp <= deadline, "Execution deadline expired");
```

File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L330 )

```solidity
1129: return uint32(block.timestamp);
```

### <a id=L3>[L-2]</a> `_safeMint()` should be used rather than `_mint()` wherever possible.

##### Description

`_safeMint()` is used along with [IERC721Receiver](https://docs.openzeppelin.com/contracts/2.x/api/token/erc721#IERC721Receiver) that checks if you are sending the minted token to a Contract that is capable to manage NFTs or not. This is to prevent tokens from being lost.

##### Recommendation

Change from` _mint` to `_safeMint`.

##### *Instances (1):*

File: [2023-01-drips/src/NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L68 )

```solidity
68: _mint(to, tokenId);
```

### <a id=L4>[L-3]</a> Lock pragmas to specific compiler version.

##### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

##### Recommendation

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. [solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

##### *Instances (8):*

File: [2023-01-drips/src/AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

File: [2023-01-drips/src/Caller.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

File: [2023-01-drips/src/DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

File: [2023-01-drips/src/ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

File: [2023-01-drips/src/Managed.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

File: [2023-01-drips/src/NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

File: [2023-01-drips/src/Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L2 )

```solidity
2: pragma solidity ^0.8.17;
```

### Non-critical Issues

### <a id=NC1>[NC-1]</a> Constants should be defined rather than using magic numbers

##### Description

Even [`assembly`](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

##### Recommendation

You should declare the variable instead of magic number.

##### *Instances (9):*

File: [2023-01-drips/src/AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L41 )

```solidity
41: return (uint256(driverId) << 224) | uint160(userAddr);
```

File: [2023-01-drips/src/Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L35 )

```solidity
66: config = (config << 160) | amtPerSec_;
67: config = (config << 32) | start_;
68: config = (config << 32) | duration_;
74: return uint32(DripsConfig.unwrap(config) >> 224);
79: return uint160(DripsConfig.unwrap(config) >> 64);
84: return uint32(DripsConfig.unwrap(config) >> 32);
```

File: [2023-01-drips/src/ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L37 )

```solidity
37: return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_counterSlot).value;
```

File: [2023-01-drips/src/NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L51 )

```solidity
51: return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_mintedTokensSlot).value;
```
