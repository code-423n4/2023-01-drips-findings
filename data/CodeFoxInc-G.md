
# Gas Optimization

## [G-01] use custom error to save gas

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L47](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L47)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L54](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L54)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L61](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L61)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L67](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L67)

The code above can be converted to using customized error to save gas use. This can be applied to all code base. 

Custom errors from Solidity 0.8.4 are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met). You can read [this](https://blog.soliditylang.org/2021/04/21/custom-errors/).

### Recommendation

I suggest replacing revert strings with custom errors.

## [G-02] Use unchecked bracket for gas saving

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L632](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L632)

The code below can be put into uncheck bracket to save gas. Because of the `require` code before it, there is no risk of overflow. 

```solidity
require(totalBalances[erc20] + amt <= MAX_TOTAL_BALANCE, "Total balance too high");
totalBalances[erc20] += amt; // @audit - can use unchecked math here to save
```

This is an [example](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/785f65183c3ca6826cb0d4c160f65f4f92e33460/contracts/token/ERC20/ERC20.sol#L257-L260) of how to use it. 

## [G-03] Use `!=0` instead of `>0`

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/AddressDriver.sol#L132](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/AddressDriver.sol#L132)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Splits.sol#L237](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Splits.sol#L237)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/NFTDriver.sol#L69](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/NFTDriver.sol#L69)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/NFTDriver.sol#L86](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/NFTDriver.sol#L86)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/ImmutableSplitsDriver.sol#L67](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/ImmutableSplitsDriver.sol#L67)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L245](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L245)

It is recommended that using `!= 0` is more gas efficient than `> 0` .

Proof: While it may seem that `> 0` is cheaper than `!=`, this is only true without the optimizer enabled and outside a require statement. If you enable the optimizer at 10k AND you're in a `require` statement, this will save gas. You can see this tweet for more proofs: [https://twitter.com/gzeon/status/1485428085885640706](https://twitter.com/gzeon/status/1485428085885640706)

## [G-04] **Make Function `external` instead of `public`**

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L270-L276](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L270-L276)

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L297-L303](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L297-L303)

Functions can be set as `external` to optimize the gas cost.