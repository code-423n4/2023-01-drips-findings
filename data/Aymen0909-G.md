# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 2 |
| 2  | Use `unchecked` blocks to save gas  |  5 |
| 3  | `storage` variable should be cached into `memory` variables instead of re-reading them  |  1 |
| 4  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  |  4 |
| 5  | `require()` strings longer than 32 bytes cost extra gas  |  5 |
| 6  | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops |  14  |


## Findings

### 1- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 2 instances of this issue :

```
File: Caller.sol

88      mapping(address => EnumerableSet.AddressSet) internal _authorized;
90      mapping(address => uint256) public nonce;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct Sender {
    uint256 nonce;
    EnumerableSet.AddressSet _authorized;
}
    
mapping(address => Sender) private _senders;
```


### 2- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 5 instances of this issue:

File: Drips.sol [Line 283-284](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283-L284)
```
receivableCycles = toCycle - fromCycle - maxCycles;
toCycle -= receivableCycles;
```

The first operation cannot underflow due to the if check statement that preceeds it and the second operation cannot underflow because we will always have `toCycle > receivableCycles` from the first operation, so both operations should be marked as `unchecked`. 

File: DripsHub.sol [Line 632](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632)
```
totalBalances[erc20] += amt;
```

The above operation cannot overflow due to the check that preceeds [Line 283](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L631) so it should be marked as `unchecked`. 

File: Splits.sol [Line 130-131](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L130-L131)
```
splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);
collectableAmt = amount - splitAmt;
```

The first operation cannot overflow due because we always have `_TOTAL_SPLITS_WEIGHT > splitsWeight` when setting the split, and the second operation cannot underflow because we will always have `amount > splitAmt` from the first operation, so both operations should be marked as `unchecked`. 

### 3- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There is 1 instance of this issue :

File: Drips.sol [Line 540-542](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L540-L542)

In the code linked above the value of `state.updateTime` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.
 

### 4- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 4 instances of this issue:

```
File: Drips.sol

253      amtDeltas[toCycle].thisCycle += finalAmtPerCycle;

File: DripsHub.sol

632      totalBalances[erc20] += amt;
636      _dripsHubStorage().totalBalances[erc20] -= amt;

File: Splits.sol

168      balance.collectable += collectableAmt;
```

### 5- `require()` strings longer than 32 bytes cost extra gas :

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas.

There are 5 instances of this issue :

```
File: Drips.sol

454     require(dripsHash == 0, "Drips history entry with hash and receivers");
540     require(timestamp >= state.updateTime, "Timestamp before last drips update");
798     require(amtPerSec != 0, "Drips receiver amtPerSec is zero");

File: NFTDriver.sol

41      require(
            _isApprovedOrOwner(_msgSender(), tokenId),
            "ERC721: caller is not token owner or approved"
        );

File: Splits.sol

239     require(prevUserId < userId, "Splits receivers not sorted by user ID");
```

### 6- `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops :

There are 14 instances of this issue:
 
```
File: Caller.sol

196     for (uint256 i = 0; i < calls.length; i++) 

File: Drips.sol

247     for (uint32 cycle = fromCycle; cycle < toCycle; cycle++)
287     for (uint32 cycle = fromCycle; cycle < toCycle; cycle++)
358     for (uint256 i = 0; i < squeezedNum; i++)
422     for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++)
458     for (uint256 i = 0; i < dripsHistory.length; i++)
490     for (; idx < receivers.length; idx++)
563     for (uint256 i = 0; i < receivers.length; i++)
664     for (uint256 i = 0; i < newReceivers.length; i++)

File: DripsHub.sol

613     for (uint256 i = 0; i < userMetadata.length; i++)

File: ImmutableSplitsDriver.sol

61      for (uint256 i = 0; i < receivers.length; i++)

File: Splits.sol

127     for (uint256 i = 0; i < currReceivers.length; i++)
158     for (uint256 i = 0; i < currReceivers.length; i++)
231     for (uint256 i = 0; i < receivers.length; i++)
```