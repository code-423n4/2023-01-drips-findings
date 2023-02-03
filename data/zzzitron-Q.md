## Summary

Risk | Title
---|---
NC00 | `driverId` offset is hardcoded in the driver contracts
NC01 | Wrong and confusing naming of variable in AddressDriver.t.sol
NC02 | Unit tests are missing comments
L01 | Incomplete/missing NatSpec tags in function comments
L02 | Extensive usage of slither-disable, posing a risk, and also causing source code deterioration
L03 | DOS condition when registering type(uint32).max number of drivers
L04 | Wrong and misleading comment in DripsHub.sol
L05 | DOS condition when adding max amount of tokens to drips balance
L06 | Wrong and misleading comment, plus possible risk for future code changes
L07 | Bad english in comment in DripsHub.sol, can be misleading
L08 | Certain ERC20 tokens like UNI and COMP are not compatible with the AddressDriver contract
L09 | Wrong parameter name used for NatSpec tag in function comment
L10 | Wrong NatSpec tag used in function comment
## Non-critical

### NC00: `driverId` offset is hardcoded in the driver contracts

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L37
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L51
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L41
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L67

In DripsHub.sol there is a constant `DRIVER_ID_OFFSET`, but in ImmutableSplitsDriver, NFTDriver and AddressDriver the offset is just hardcoded. Instead of using the hardcoded value, use the constant `DRIVER_ID_OFFSET`.


```solidity
// DripsHub
67    uint256 public constant DRIVER_ID_OFFSET = 224;
```

```solidity
// NFTDriver
// Don't use hardcoded value 224 here to shift the driverId. Should use DRIVER_ID_OFFSET instead of 224:
51        return (uint256(driverId) << 224) + StorageSlot.getUint256Slot(_mintedTokensSlot).value;
```

In the NFTDriver, the hardcoded value 224 is used for the offset of the `driverId`, see code snippet above. Also in AddressDriver and ImmutableSplitsDriver a hardcoded value is used for the offset of the `driverId`. It would be better to use the constant `DRIVER_ID_OFFSET` instead.

### NC01: Wrong and confusing naming of variable in AddressDriver.t.sol

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/test/AddressDriver.t.sol#L41

```solidity
// AddressDriver.t.sol
41        uint32 nftDriverId = dripsHub.registerDriver(address(this));
42        AddressDriver driverLogic = new AddressDriver(dripsHub, address(caller), nftDriverId);
```

The variable name `nftDriverId` on line 41 from the code snippet above is confusing, and should be renamed to `addressDriverId`. Fixed code example:

```solidity
41        uint32 addressDriverId = dripsHub.registerDriver(address(this));
42        AddressDriver driverLogic = new AddressDriver(dripsHub, address(caller), addressDriverId);
```

### NC02: Unit tests are missing comments

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/test/AddressDriver.t.sol#L92-L122

The above reference to AddressDriver.t.sol#L92-L122 is just an example. In general the existing unit tests are lacking comments. Without comments one has to read the source code to understand what is really done in the specific unit test.

For example the referenced unit test `AddressDriver.t.sol:testSetDrips` is both adding drips balance via the `setDrips` function (line 101) and then withdrawing the balance via another call to the `setDrips` function (line 116).

Some comments could be added to the unit test to help approaching it and help understanding it easier. Example/Suggestion:

```solidity
// AddressDriver.t.sol
89:     // This unit test checks the setDrips function with a single receiver.
90:     // 1) First setDrips is called to increase the drips balance.
91:     // 2) Second setDrips is called again to withdraw the drips balance.
92:     function testSetDrips() public {
```

Without comments, it will be much harder in the future to approach the unit tests, reading them or adding new unit tests. For example when you get a bug report and you want to check your unit tests for the bug reported. Without comments, you might have to read through a lot of source code again to understand what the unit tests are actually doing. This applies especially for the more complex unit tests, which would really benefit from having some comments, making life easier in the future in the case of a bug report etc.

Another example is when you want to add additional unit tests, you have to read again the source code to identify which existing unit tests you already have, to avoid creating duplicate unit tests that are testing the same use case etc.

## Low

### L01: Incomplete/missing NatSpec tags in function comments

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L170

The function AddressDriver._transferFromCaller is missing the NatSpac tags @notice and @params.
Other functions inside AddressDriver.sol are featuring these NatSpac tags.
As an example AddressDriver._setSplits (line 158) has comments on line 149-157 about @notice and @param, describing what this function does and describing the params of the function.

This is not very consistent and it makes it harder to understand what certain functions are doing, because the comments are missing.

Moreover Etherscan (etherscan.io) uses this information from the NatSpac tags, but if they are missing, it will not work for Etherscan.

Here are more missing NatSpac tags as well:

Missing @notice, @param and @return tags:

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L202

Missing @param tag:

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L82-L84

Missing @notice and @param tags:

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L124

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L629

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L635

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L285

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L98

Missing @notice tag:

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L158

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L40

Missing @notice and @return tags:

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L294

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L300

Missing @return tags:

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L985

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L970

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1061


### L02: Extensive usage of slither-disable, posing a risk, and also causing source code deterioration

When searching in your source code for the term "slither-disable" there are 43 results. This basically means that you shut down 43 warnings from your static code analyzer, which is a very high number.

Instead of just shutting down the warnings with "slither-disable", it is recommended to resolve as many warnings as possible, since they all pose a potential risk. The potential risk is either having a negative effect or even a bug already in the present, or latest in the future.


Here is a quote from the broken window theory:

> We've seen clean, functional systems deteriorate pretty quickly once windows start breaking. There are other factors that can contribute to software rot, and we'll touch on some of them elsewhere, but neglect accelerates the rot faster than any other factor.
(Source: https://blog.codinghorror.com/the-broken-window-theory/)


Don't let your source code deteriorate. You already have 43 broken windows. Try to fix as many as possible, getting rid of "slither-disable".

### L03: DOS condition when registering type(uint32).max number of drivers

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L137

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L99

If type(uint32).max drivers are registered in the DripsHub contract, then line 137 in the DripsHub contract will revert, because on line 99 the driverAddresses mapping can hold a maximum of type(uint32).max driver addresses. If this maximum is reached, line 137 always reverts, leading to a DOS condition when someone wants to register another driver.

### L04: Wrong and misleading comment in DripsHub.sol

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L173

The comment suggests that the function DripsHub.totalBalance (line 181) returns the total amount of the ERC20 token stored in DripsHub.

So, according the the comment, line 182 should read:

```solidity
// DripsHub
182        return erc20.balanceOf(address(this));
```

But line 182 actually reads:

```solidity
// DripsHub
182        return _dripsHubStorage().totalBalances[erc20];
```
This means that the tracked total balance of the erc20 token stored in the DripsHub contract is returned. The total amount currently stored in the DripsHub contract is not returned, otherwise erc20.balanceOf(address(this)) would have to be used on line 182.

Using erc20.balanceOf would be wrong, because anybody can privately send tokens to the DripsHub contract, but of course you don't want to track these "wild" tokens. You only want to track tokens that were sent through the business logic functions of the DripsHub contract, like for example on line 533 where tokens are sent to the DripsHub contract to increase the drips balance.

Thus, the comment on line 173 is misleading. DripsHub.totalbalance (line 181) is not returning the total amount of the ERC20 token stored in DripsHub. Instead the comment on line 173 should read:

```solidity
// DripsHub
173    /// @notice Returns the total amount currently tracked in DripsHub of the given token.
```

### L05: DOS condition when adding max amount of tokens to drips balance

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L510

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L631

A malicious actor sends type(int128).max tokens to the DripsHub contract, by using the function DripsHub.setDrips (line 510) and passing the value to the `balanceDelta` parameter of the function. The goal of the malicious user is to make sure that line 631 in the DripsHub contract will revert in the future, whenever another user wants to add drips balance. Line 631 in the DripsHub contract reverts if MAX_TOTAL_BALANCE is exceeded, which is type(int128).max, so the malicious user adds drips balance so that the total balance of the DripsHub contract for this token is exactly MAX_TOTAL_BALANCE. This means that no more drips balance can be added, leading to a DOS condition, whenever another user wants to add drips balance via the DripsHub.setDrips function, because line 631 will always revert.

The malicious actor can also check the current totalBalance of the token in the DripsHub contract and just send enough tokens to reach MAX_TOTAL_BALANCE:
```solidity
// MaliciousContract

// 1. Calculate the amount of tokens needed to create a DOS condition
uint128 requiredTokensToDOS = type(int128).max - dripsHub.totalBalance(erc20);

// 2. Send the amount of tokens to the DripsHub contract to create the DOS condition
dripsHub.setDrips(userId, erc20, receivers, requiredTokensToDOS, receivers, 0, 0);

// 3. DOS achieved, nobody can add any more of the erc20 token to the DripsHub contract via the DripsHub.setDrips function.
```

In the malicous contract snippet above, `requiredTokensToDOS` are calculated and then sent to the DripsHub contract via the DripsHub.setDrips function. This achieves the DOS condition, because every further attempt calling the setDrips function with a positive `balanceDelta` param will revert on line 631 of the DripsHub contract, because the maximum allowed balance is exceeded.

A malicious actor could also frontrun other actors/users to intentionally create DOS conditions like the described above, whenever another actor/user wants to add drips balance via the DripsHub.setDrips function.

### L06: Wrong and misleading comment, plus possible risk for future code changes

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L69-L70

The comment suggests that `MAX_TOTAL_BALANCE` is the minimum of `_MAX_TOTAL_DRIPS_BALANCE` and `_MAX_TOTAL_SPLITS_BALANCE`. If that would really be the case, line 70 in the DripsHub contract should read:

```solidity
// DripsHub
// Using OpenZeppelins Math library to calculate the min of the two values:
70    uint256 public constant MAX_TOTAL_BALANCE = Math.min(_MAX_TOTAL_DRIPS_BALANCE, _MAX_TOTAL_SPLITS_BALANCE);
```

But the current code line 70 in the DripsHub contract just hardcoded assigns `_MAX_TOTAL_DRIPS_BALANCE` to `MAX_TOTAL_BALANCE`. It doesn't use the minimum value of `_MAX_TOTAL_DRIPS_BALANCE` and `_MAX_TOTAL_SPLITS_BALANCE`, like suggested in the code snippet above using Math.min or some other code to determine the minimum value.

In the future, if one or both values of `_MAX_TOTAL_DRIPS_BALANCE` and `_MAX_TOTAL_SPLITS_BALANCE` are changed, `MAX_TOTAL_BALANCE` might not return the minimum of `_MAX_TOTAL_DRIPS_BALANCE` and `_MAX_TOTAL_SPLITS_BALANCE` any more, because it just hardcoded returns `_MAX_TOTAL_DRIPS_BALANCE` which might then be higher than `_MAX_TOTAL_SPLITS_BALANCE`, thus returning not the minimum any more. This poses a certain risk.

The solution to fix this risk is, that the real minimum should be assigned to `MAX_TOTAL_BALANCE`, as in the code snippet above via the Math.min function. This also fixes the wrong comment.

### L07: Bad english in comment in DripsHub.sol, can be misleading

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L446

Currently the existing comment suggests, that the user drips balance, which is not the case. What is meant in the comment is the "User's drips balance".

Current wrong comment:

```solidity
// DripsHub
// Wrong comment below, the user does not drip balance.
446    /// @notice User drips balance at a given timestamp
```

The comment should read:

```solidity
// DripsHub
446    /// @notice User's drips balance at a given timestamp
```


### L08 | Certain ERC20 tokens like UNI and COMP are not compatible with the AddressDriver contract

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L174

Certain ERC20 tokens like UNI and COMP will revert on approve with a too large amount.

See https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers

These ERC20 tokens cannot be used with the AddressDriver contract, because upon calling AddressDriver.setDrips and AddressDriver.give functions, AddressDriver._transferFromCaller will be invoked which will revert on line 174, because UNI and COMP only allow a maximum of type(uint96).max to be approved.



### L09 | Wrong parameter name used for NatSpec tag in function comment

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1060

On line 1060 a wrong param name "prev" is used. The param name should correctly read "next":

```solidity
// Drips
1060    /// @param next The next receiver
```


### L10 | Wrong NatSpec tag used in function comment

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1092

Line 1092 uses the NatSpec tag @param for the function return variable `amt`. Since `amt` is the name of the return variable, the correct NatSpec tag to use is @return.

Should read:

```solidity
// Drips
1092:     /// @return amt The dripped amount
```