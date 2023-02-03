# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | The function `callSigned` doesn't revert if `sender == address(0)` | Low | 1 |
| 2 | Same driver address can be registred twice with different id | Low | 1 |
| 3 | Named return variables not used anywhere in the functions | NC | 20 |
| 4 | Use scientific notation | NC | 5 |


## Findings

### 1- The function `callSigned` doesn't revert if `sender == address(0)` :

In both `Caller` contract the `callSigned` function allow users to excute calls using already signed messages and uses the `_call` lethod to do so, but the function does not check if the input sender address is different from the zero address and thus if for some reason we have `sender == address(0)` the function will not revert and excutes the call with `_call(address(0), to, data, msg.value)`. This can impact the functions that receive the call.

#### Risk : Low

#### Proof of Concept

Issue occurs in the instance below :

File: Caller.sol [Line 164-183](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L164-L183)
```
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
    /** @audit
    The function does not check if sender != address(0)
    */
    require(signer == sender, "Invalid signature");
    return _call(sender, to, data, msg.value);
}
```

#### Mitigation

To avoid this issue i recommend to add a non zero address check on the input `sender` address

### 2- Same driver address can be registred multiple times with different ids :

#### Risk : Low 

The `register` function allow a unique driver address to be registered multiple times with different ids, in the current code logic this does not have an impact on the protocol as the driverId of each driver can not be chnaged after deployment but this issue must be taken into account for future development if the protocol for some reason uses the driverId associated with a given driver address.

### 3- Named return variables not used anywhere in the function :

When Named return variable are declared they should be used inside the function instead of the return statement or if not they should be removed to avoid confusion.

#### Risk : NON CRITICAL

#### Proof of Concept

Instances include:

File: AddressDriver.sol [Line 40](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L40)
```
returns (uint256 userId)
```

File: Drips.sol [Line 303](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L303)

```
returns (uint32 cycles)
```

File: Drips.sol [Line 475](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L475)

```
returns (uint128 squeezedAmt)
```

File: Drips.sol [Line 511](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L511)

```
returns (
    bytes32 dripsHash,
    bytes32 dripsHistoryHash,
    uint32 updateTime,
    uint128 balance,
    uint32 maxEnd
)
```

File: Drips.sol [Line 742](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L742)

```
returns (bool isEnough)
```

File: Drips.sol [Line 795](https://github.com/code-953n4/2023-01-drips/blob/main/src/Drips.sol#L795)

```
returns (uint256 newConfigsLen)
```

File: Drips.sol [Line 837](https://github.com/code-953n4/2023-01-drips/blob/main/src/Drips.sol#L837)

```
returns (bytes32 dripsHash)
```

File: DripsHub.sol [Line 145](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L145)
```
returns (address driverAddr)
```

File: DripsHub.sol [Line 160](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L160)
```
returns (uint32 driverId)
```

File: DripsHub.sol [Line 169](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L169)
```
returns (uint32 cycleSecs_)
```

File: DripsHub.sol [Line 181](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L181)
```
returns (uint256 balance)
```

File: ImmutableSplitsDriver.sol [Line 36](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L36)
```
returns (uint256 userId)
```

File: Managed.sol [Line 105](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L105)
```
returns (bool isAddrPauser)
```

File: Managed.sol [Line 112](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L112)
```
returns (address[] memory pausersList)
```

File: Managed.sol [Line 117](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L117)
```
returns (bool isPaused)
```

File: NFTDriver.sol [Line 50](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L50)
```
returns (uint256 tokenId)
```

File: Splits.sol [Line 106](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L106)
```
returns (uint128 amt)
```

File: Splits.sol [Line 176](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L176)
```
returns (uint128 amt)
```

File: Splits.sol [Line 262](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L262)
```
returns (bytes32 currSplitsHash)
```

File: Splits.sol [Line 273](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L273)
```
returns (bytes32 receiversHash)
```

#### Mitigation

Either use the named return variables inplace of the return statement or remove them.

### 4- Use scientific notation :

When using multiples of 10 you shouldn't use decimal literals or exponentiation (e.g. 1000000, 10**18) but use scientific notation for better readability.

#### Risk : NON CRITICAL

#### Proof of Concept

Instances include:

File: Splits.sol [Line 22](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L22)

```
uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
```

File: Drips.sol [Line 110](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L110)

```
uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
```

#### Mitigation

Use scientific notation for the instances aforementioned.