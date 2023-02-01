# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       | Wrong check for duplicate receivers|1|
|L-02       | Collect doesn't send to msg.sender|1|
|L-03       | Other contract addresses can only be set once|1|
|L-04       | Critical address change should be a two step process|1|
## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Not using the named return variables in a function|11|
|N-02       | Unspecific compiler version pragma|1|
|N-03       | Wrong comments|1|
|N-04       | Event is missing indexed fields|2|
# Details
# Low Risk
## [L-01] Wrong check for duplicate receivers
The _assertSplitsValid function checks on L238 if there are duplicate receivers. However it only checks the last and the current receiver. Let's say I have a list of receivers(user1, user2, user1). It will check if user1 is equal to user2 and sees it has valid. It will not check if user1 is equal to the first position in the list only to the previous receiver. But because of the check underneath (prevUserId < userId) it's not possible to do it this way and have duplicate users. So the require statement `prevUserId != userId` can be ommitted.
[Splits.sol#L238](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L238)
```solidity
        uint256 prevUserId;
        for (uint256 i = 0; i < receivers.length; i++) {
            SplitsReceiver memory receiver = receivers[i];
            uint32 weight = receiver.weight;
            require(weight != 0, "Splits receiver weight is zero");
            totalWeight += weight;
            uint256 userId = receiver.userId;
            if (i > 0) {
                require(prevUserId != userId, "Duplicate splits receivers");
                require(prevUserId < userId, "Splits receivers not sorted by user ID");
            }
            prevUserId = userId;
            emit SplitsReceiverSeen(receiversHash, userId, weight);
        }
```
## [L-02] Collect doesn't send to msg.sender
The collect function sends the funds to the transferTo parameter and not to msg.sender. If the user implements a wrong transferTo address, all the funds are lost. It might better to just transfer to msg.sender
[AddressDriver.sol#L60-L63](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L60-L63)
```solidity
    function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
        amt = dripsHub.collect(callerUserId(), erc20);
        erc20.safeTransfer(transferTo, amt);
    }
```

## [L-03] Other contract addresses can only be set once
Other contract addresses are only set once in the constructor. This makes it so there is no room for error and can lead to redeployment
- [AddressDriver.sol#L30-L35](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30-L35)
- [ImmutableSplitsDriver.sol#L28-L32](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28-L32)
- [NFTDriver.sol#L32-L38](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32-L38)
## [L-04] Critical address change should be a two step process
[Managed.sol#L84-L86](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84-L86)
# Non critical
## [N-01$] Not using the named return variables in a function
- [ImmutableSplitsDriver.sol#L37](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L37): return variable userId is never used
- [Drips.sol#L470-L486](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L470-L486): return variable squeezedAmt is never used
- [Drips.sol#L508-L522](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L508-L522): all the named return variables are unused
- [Drips.sol#L538](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L538): return variable balance is never used
- [Drips.sol#L737-L762](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L737-L762): return variable isEnough is never used
- [Drips.sol#L792-L807](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L792-L807): return variable newConfigsLen is never used
- [Drips.sol#L815-L826](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L815-L826): all the named return variables are unused
- [Drips.sol#L834-L843](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L834-L843): return variable dripsHash is never used
- [Drips.sol#L852-L859](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L852-L859): return variable dripsHistoryHash is never used
- [Splits.sol#L106-L108](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L106-L108): return variable amt is never used
- [Splits.sol#L176-L178](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L176-L178): return variable amt is never used
## [N-02] Unspecific compiler version pragma
Contracts should be deployed with the same compiler version and flags that they have been tested the most with. If you lock the pragme there is no chance of deploying with a version that contains bugs. On this protocol, the pragma is the latest version but depending on when it get's deployed on the mainnet it can be an issue.
## [N-03] Wrong comments
When executing the give function the comment on [AddressDriver.sol#L66](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L66) says The receiver can collect them immediately. However it will be added to the splittable balance. The comment on [DripsHub.sol#L398](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L398) is correct.
## [N-04] Event is missing indexed fields
When an event is missing indexed fields it can be hard for off-chain scripts to efficiently filter those events.

[Managed.sol#L34](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L34) and [Managed.sol#L38](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L38)
