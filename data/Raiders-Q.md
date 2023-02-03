### 1. Variable Declared but never used

Description: 
- The below-mentioned list of variables was found to be declared internal constant which causes gas but were never used.

#### POC:
- ` _MAX_TOTAL_DRIPS_BALANCE`
- `_MAX_TOTAL_SPLITS_BALANCE`

Github link:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L112
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L69

Mitigation:
- Remove dead code/variables if not used/needed

### 2. Gas Theft possible due to Unchecked array length

Description:
Ethereum is a very resource-constrained environment. Prices per computational step are orders of magnitude higher than with centralized providers. Moreover, Ethereum miners impose a limit on the total number of Gas consumed in a block. If `array.length` is large enough, the function exceeds the block gas limit, and transactions calling it will never be confirmed.

`for (uint256 i = 0; i < array.length ; i++) { cosltyFunc(); }`
This becomes a security issue, if an external actor influences `array. Length`.


E.g., if an array enumerates all registered addresses, an adversary can register many addresses, causing the problem described above.

#### PoC

```
bool pickCurr = currIdx < currReceivers.length;
```
Github code:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L883
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L890
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196
- https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61

Remediation:
- Either explicitly or just due to normal operation, the number of iterations in a loop can grow beyond the block gas limit, which can cause the complete contract to be stalled at a certain point. Therefore, loops with a bigger or unknown number of steps should always be avoided.

### 3. INDEXED KEYWORDS IN EVENTS NOT USED
- *Severity*: Low

- Impact: Events are essential for tracking off-chain data and when the event parameters are indexed, they can be used as filter options, which will help getting only the specific data instead of all the logs.

#### POC :
```
    event Paused(address pauser);
````
*Github*
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L34
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L38

### 4. Excessive function returns must use Struct to return variables
- *Severity* : low

- *Impact: The function `_receiveDripsResult` was detected to be returning multiple values.
Consider using a struct instead of multiple return values for the function. It can improve code readability

#### PoC:
```
    function _receiveDripsResult(uint256 userId, uint256 assetId, uint32 maxCycles)
        internal
        view
        returns (
            uint128 receivedAmt,
            uint32 receivableCycles,
            uint32 fromCycle,
            uint32 toCycle,
            int128 amtPerCycle
        )
```

Github:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L270
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L392
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L508
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L118

*Remediation:*
- Use struct for returning multiple values inside a function, which returns several parameters and improves code readability.

### 5. FUNCTION SHOULD BE EXTERNAL

*Impact:*
- A function with public visibility modifier was detected that is not called internally.
- public and external differs in terms of gas usage. The former use more than the latter when used with large arrays of data. This is due to the fact that Solidity copies arguments to memory on a public function while external read from calldata which a cheaper than memory allocation.

#### POC
```
    function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
        _assertCallerIsDriver(driverId);
        _dripsHubStorage().driverAddresses[driverId] = newDriverAddr;
        emit DriverAddressUpdated(driverId, msg.sender, newDriverAddr);
    }
```
Github
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L152
 
Remediation
- If you know the function you create only allows for external calls, use the external visibility modifier instead of public. It provides performance benefits and you will save on gas.


