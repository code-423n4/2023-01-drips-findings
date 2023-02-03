# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Unchecked Cast May Overflow**](#1-Unchecked-Casts-May-Overflow)
2. [**Low block.timestamp Data Size**](#2-Low-blocktimestamp-Data-Size)
3. [**Consider Known Library Use**](#3-Consider-Known-Library-Use)

[**Non-Critical**](#Non-Critical)
1. [**Whitespace In Operation**](#1-Whitespace-In-Operation)
2. [**Order of Functions Not Compliant With Solidity Docs**](#2-Order-of-Functions-Not-Compliant-With-Solidity-Docs)
3. [**Contract Layout Voids Solidity Docs**](#3-Contract-Layout-Voids-Solidity-Docs)
4. [**Inconsistent Named Returns**](#4-Inconsistent-Named-Returns)
5. [**`transferable` Spelling**](#5-transferable-Spelling)
6. [**American Non-American Spelling**](#6-American-Non-American-Spelling)
7. [**Named Returns Not Returning Named Variable**](#7-Named-Returns-Not-Returning-Named-Variable)

# Low Severity

## 1. Unchecked Casts May Overflow

As of Solidity 0.8 overflows are handled automatically; however, not for casting. For example `uint32(4294967300)` will result in `4` without reversion. Consider using OpenZepplin's SafeCast library.

*/src/Drips.sol*

```solidity
74:	return uint32(DripsConfig.unwrap(config) >> 224);
79:	return uint160(DripsConfig.unwrap(config) >> 64);
84:	return uint32(DripsConfig.unwrap(config) >> 32);
89:	return uint32(DripsConfig.unwrap(config));
289:	receivedAmt += uint128(amtPerCycle);
366:	_addDeltaRange(state, cycleStart, cycleStart + 1, -int256(amt * _AMT_PER_SEC_MULTIPLIER));
497:	return uint128(amt);
572:	balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
636:	newBalance = uint128(currBalance + realBalanceDelta);
692:	return uint32(enoughEnd);
697:	return uint32(notEnoughEnd);
719:	return uint32(end);
825:	return (val >> 64, uint32(val >> 32), uint32(val));
917:	int256 amtPerSec = int256(uint256(currRecv.config.amtPerSec()));
938:	int256 amtPerSec = int256(uint256(currRecv.config.amtPerSec()));
946:	int256 amtPerSec = int256(uint256(newRecv.config.amtPerSec()));
997:	uint40 end = uint40(start) + receiver.config.duration();
1012:	return (start, uint32(end));
1044:	int256 amtPerSecMultiplier = int256(_AMT_PER_SEC_MULTIPLIER);
1045:	int256 fullCycle = (int256(uint256(_cycleSecs)) * amtPerSec) / amtPerSecMultiplier;
1047:	int256 nextCycle = (int256(timestamp % _cycleSecs) * amtPerSec) / amtPerSecMultiplier;
1048:	AmtDelta storage amtDelta = amtDeltas[_cycleOf(uint32(timestamp))];
1053:	amtDelta.thisCycle += int128(fullCycle - nextCycle);
1054:	amtDelta.nextCycle += int128(nextCycle);
1129:	return uint32(block.timestamp);
```

*/src/DripsHub.sol*

```solidity
119:	uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
521:	_increaseTotalBalance(erc20, uint128(balanceDelta));
533:	erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));
535:	_decreaseTotalBalance(erc20, uint128(-realBalanceDelta));
536:	erc20.safeTransfer(msg.sender, uint128(-realBalanceDelta));
643:	return uint160(address(erc20));
```

*/src/Splits.sol*

```solidity
130:	splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);
161:	uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);
```

*/src/NFTDriver.sol*

```solidity
198:	_transferFromCaller(erc20, uint128(balanceDelta));
204:	erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
```

*/src/AddressDriver.sol*

```solidity
41:	return (uint256(driverId) << 224) | uint160(userAddr);
133:	_transferFromCaller(erc20, uint128(balanceDelta));
145:	erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
```

## 2. Low block.timestamp Data Size

Throughout the codebase there are data size assumptions and casting of `block.timestamp` that can be seen in the issues above. A direct example is [L1129](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1129) of Drips.sol.

As of now `uint32(block.timestamp)` will overflow when the `block.timestamp` is greater than `type(uint32).max` := 4294967295. [`block.timestamp` returns the current seconds since unix epoch](https://docs.soliditylang.org/en/v0.8.13/units-and-global-variables.html). A unix time of 4294967295 would imply a date and time of **Sunday, 7 February 2106 06:28:15 GMT** (can be verified [here](https://www.epochconverter.com/)). In approximetly 83 years the codebase will malfunction and likely put user funds at risk given how much the protocol is reliant on timekeeping. 

Consider using a larger data size for `block.timestamp` (EX. `uint64`, `uint128`, ...).

## 3. Consider Known Library Use

The [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L117) contract has pause functionality that is self-written. Consider using OpenZepplins pausable contract. For security it is best to use battle-tested code when possible.

# Non-Critical

## 1. Whitespace In Operation

Exponentiation should not have trailing and leading spaces as seen in the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#variable-declarations).

*/src/Drips.sol*

```solidity
185:	mapping(uint256 => uint32[2 ** 32]) nextSqueezed;
357:	uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
419:	uint32[2 ** 32] storage nextSqueezed =
```

## 2. Order of Functions Not Compliant With Solidity Docs

The Solidity Style Guide suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol): internal functions are positioned after private functions.
* [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol): public functions are positioned after internal functions.
* [Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol): internal functions are positioned after private functions.
* [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol): public functions are positioned after internal functions.
* [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol): public functions are positioned after internal functions.

## 3. Contract Layout Voids Solidity Docs

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) suggests the following contract layout order: Type declarations, State variables, Events, Modifiers, Functions.

The following contracts are not compliant (examples are only to prove the layout are out of order NOT a full description): 

* [Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol): Type declarations are positioned after State.
* [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol): State is positioned after Events.
* [Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol): State is positioned after Events.
* [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol): Modifiers are positioned after Constructor (functions).
* [Managed.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol): State is positioned after Events.

## 4. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. The following contracts only have named returns (EX. `returns(uint256 foo)`): [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol), [Splits.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol), [Caller.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol), [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol), and [ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol).
2. No contracts only have non-named returns (EX. `returns(uint256)`).
3. The following contracts have both: [Drips.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol), [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol), and [Managed.sol](https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol).

## 5. `transferable` Spelling

`transferable` is misspelled as `transferrable` throughout the codebase. Consider fixing all spelling mistakes.

*/src/DripsHub.sol*

* [L176](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L176), [L191](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L191), [L208](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L208), [L230](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L230), [L257](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L257), [L289](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L289), [L312](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L312), [L347](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L347), [L367](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L367), [L381](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L381), [L404](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L404), [L423](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L423), [L450](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L450), [L475](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L475).

*/src/NFTDriver.sol*

* [L103](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L103), [L128](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L128), [L150](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L150).

*/src/AddressDriver.sol*

* [L54](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L54), [L71](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L71), [L86](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L86).

## 6. American Non-American Spelling

Words in the codebase have both non-american and american spelling. Consider sticking to a single grammar.

* `fulfil` is found in [Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L910) [twice](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1058) (spelled `fulfill` in america)
* `behavior` is found in [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L49) (spelled `behaviour` outside of america)

## 7. Named Returns Not Returning Named Variable

For most functions in the cobebase named returns are used; however, the named variable is not used. Using named returns may help developers understand what is being returned, but this should be in a @return tag. There is no need to confuse developers.

*/src/Drips.sol*

* [`_receivableDripsCycles`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L300), [`_squeezedAmt`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L470), [`_dripsState`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L508), [`_balanceAt`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L533), [`_calcMaxEnd`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L680), [`_isBalanceEnough`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L737), [`_addConfig`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L792), [`_getConfig`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L815), [`_hashDrips`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L834), [`_hashDripsHistory`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L852), [`_dripsRangeInFuture`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L970), [`_dripsRange`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L985), [`_cycleOf`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1120), [`_currTimestamp`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1128), [`_currCycleStart`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1134).

*/src/DripsHub.sol*

* [`driverAddress`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L145), [`nextDriverId`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L160), [`cycleSecs`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L169), [`totalBalance`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L181), [`receivableDripsCycles`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L196). [`splittable`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L317), [`splitResult`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L328), [`split`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L355), [`collectable`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L372), [`dripsState`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L432), [`balanceAt`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L460), [`hashDrips`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L546), [`hashDripsHistory`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L557), [`splitsHash`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L587), [`hashSplits`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L595).

*/src/Splits.sol*

* [`_splittable`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L106), [`_split`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L143), [`_collectable`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L176), [`_splitsHash`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L262), [`_hashSplits`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L270).

*/src/NFTDriver.sol*

* [`nextTokenId`](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L50).

*/src/Managed.sol*

* [`isPauser`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L105), [`allPausers`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L112), [`paused`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L117), [`_erc1967Slot`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L136).

*/src/Caller.sol*

* [`isAuthorized`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L124), [`allAuthorized`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L132), [`callAs`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L144), [`callSigned`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L164), [`callBatched`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193), [`_call`](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L202).

*/src/AddressDriver.sol*

* [`calcUserId`](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L40).

*/src/ImmutableSplitsDriver.sol*

* [`nextUserId`](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L36).
