

## [L-01] Unspecific Compiler Version Pragma

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

```
2023-01-drips/src/AddressDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips/src/Caller.sol::2 => pragma solidity ^0.8.17;
2023-01-drips/src/Drips.sol::2 => pragma solidity ^0.8.17;
2023-01-drips/src/DripsHub.sol::2 => pragma solidity ^0.8.17;
2023-01-drips/src/ImmutableSplitsDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips/src/Managed.sol::2 => pragma solidity ^0.8.17;
2023-01-drips/src/NFTDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips/src/Splits.sol::2 => pragma solidity ^0.8.17;
```

## [L-02] Use of Block.timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

```
2023-01-drips/src/Caller.sol::173 => require(block.timestamp <= deadline, "Execution deadline expired");
2023-01-drips/src/Drips.sol::330 => /// Only funds dripped before `block.timestamp` can be squeezed.
2023-01-drips/src/Drips.sol::530 => /// If it's bigger than `block.timestamp`, then it's a prediction assuming
2023-01-drips/src/Drips.sol::552 => /// If it's bigger than `block.timestamp`, then it's a prediction assuming
2023-01-drips/src/Drips.sol::1129 => return uint32(block.timestamp);
2023-01-drips/src/DripsHub.sol::253 => /// Only funds dripped before `block.timestamp` can be squeezed.
2023-01-drips/src/DripsHub.sol::457 => /// If it's bigger than `block.timestamp`, then it's a prediction assuming
```

## [L-03] safeApprove() is deprecated

safeApprove() is deprecated in favor of safeIncreaseAllowance() and safeDecreaseAllowance(). If only setting the initial allowance to the value that means infinite, safeIncreaseAllowance() can be used instead

```
2023-01-drips/src/AddressDriver.sol::174 => erc20.safeApprove(address(dripsHub), type(uint256).max);
2023-01-drips/src/NFTDriver.sol::289 => erc20.safeApprove(address(dripsHub), type(uint256).max);
```

## [L-04] approve should be replaced with safeApprove or safeIncreaseAllowance() / safeDecreaseAllowance()

approve is subject to a known front-running attack. Consider using safeApprove() or safeIncreaseAllowance() or safeDecreaseAllowance() instead

```
2023-01-drips/src/NFTDriver.sol::250 => super.approve(to, tokenId);
```

## [L-05] Unsafe use of transfer()/transferFrom() with IERC20

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
2023-01-drips/src/NFTDriver.sol::282 => super.transferFrom(from, to, tokenId);
```

## [L-06] Events not emitted for important state changes

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

```
2023-01-drips/src/AddressDriver.sol::122 => function setDrips(
2023-01-drips/src/AddressDriver.sol::158 => function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
2023-01-drips/src/DripsHub.sol::510 => function setDrips(
2023-01-drips/src/DripsHub.sol::576 => function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
2023-01-drips/src/NFTDriver.sol::186 => function setDrips(
2023-01-drips/src/NFTDriver.sol::220 => function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)
2023-01-drips/src/NFTDriver.sol::272 => function setApprovalForAll(address operator, bool approved) public override whenNotPaused {
```

## [L-07] _safeMint() should be used rather than _mint() wherever possible

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
2023-01-drips/src/NFTDriver.sol::68 => _mint(to, tokenId);
```

## [L-08] Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

__gap is empty reserved space in storage that is recommended to be put in place in upgradeable contracts. It allows new state variables to be added in the future without compromising the storage compatibility with existing deployments

```
2023-01-drips/src/Managed.sol::4 => import {UUPSUpgradeable} from "openzeppelin-contracts/proxy/utils/UUPSUpgradeable.sol";
```

## [N-01] Event is missing indexed fields

Each event should use three indexed fields if there are three or more fields

```
2023-01-drips/src/Managed.sol::34 => event Paused(address pauser);
2023-01-drips/src/Managed.sol::38 => event Unpaused(address pauser);
```

## [N-02] Missing event for critical parameter change

Emitting events after sensitive changes take place, to facilitate tracking and notify off-chain clients following changes to the contract.

```
2023-01-drips/src/AddressDriver.sol::122 => function setDrips(
2023-01-drips/src/AddressDriver.sol::158 => function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
2023-01-drips/src/DripsHub.sol::510 => function setDrips(
2023-01-drips/src/DripsHub.sol::576 => function setSplits(uint256 userId, SplitsReceiver[] memory receivers)
2023-01-drips/src/NFTDriver.sol::186 => function setDrips(
2023-01-drips/src/NFTDriver.sol::220 => function setSplits(uint256 tokenId, SplitsReceiver[] calldata receivers)
2023-01-drips/src/NFTDriver.sol::272 => function setApprovalForAll(address operator, bool approved) public override whenNotPaused {
```

