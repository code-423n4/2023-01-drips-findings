## [G-01] Add `unchecked {}` for additions where the operands cannot underflow because of a previous `require()` or `if`-statement

### Description

`require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");`

*There is 1 instance of this issue:*

```solidity
File: src/DripsHub.sol

function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {
	mapping(IERC20 => uint256) storage totalBalances = _dripsHubStorage().totalBalances;
	require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");
	totalBalances[erc20] += amt;
}
```

In the `_increaseTotalBalance` function, the function checks if `totalBalances[erc20] + amt` is lesser than or equal to `MAX_TOTAL_BALANCE`. Since there is always a cap to the arithmetic operation, gas can be saved by making the operation unchecked.

[https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632)

### Recommendation

Wrapping `totalBalances[erc20] += amt;` with an unchecked block

```solidity
function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {
	mapping(IERC20 => uint256) storage totalBalances = _dripsHubStorage().totalBalances;
	require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");
	unchecked {
		totalBalances[erc20] += amt;
	}
}
```