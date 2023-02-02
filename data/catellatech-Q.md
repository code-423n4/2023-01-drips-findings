# Low Risk and Non-Critical Issues

# Unsafe ERC721 Operation(s)

#### Impact
Issue Information: [L001]
#### Findings:

```
2023-01-drips\src\NFTDriver.sol::282 => super.transferFrom(from, to, tokenId);
```

```solidity

// ...

IERC721(token).safeTransferFrom(from, to, tokenId);
```

# Unspecific Compiler Version Pragma
#### Impact
Issue Information: [L002]

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included
with multiple different versions of applications, it may be a security risk for
application implementations.

A known vulnerable compiler version may accidentally be selected or security
tools might fall-back to an older compiler version ending up checking
a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

### Example

Bad:
```solidity
pragma solidity ^0.8.0;
```

 Good:
```solidity
pragma solidity 0.8.4;
```

#### Findings:
```
2023-01-drips\src\AddressDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips\src\Caller.sol::2 => pragma solidity ^0.8.17;
2023-01-drips\src\Drips.sol::2 => pragma solidity ^0.8.17;
2023-01-drips\src\DripsHub.sol::2 => pragma solidity ^0.8.17;
2023-01-drips\src\ImmutableSplitsDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips\src\Managed.sol::2 => pragma solidity ^0.8.17;
2023-01-drips\src\NFTDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips\src\Splits.sol::2 => pragma solidity ^0.8.17;
```


# Do not use Deprecated Library Functions

#### Impact
Issue Information: [L003]

#### Findings:
```
2023-01-drips\src\AddressDriver.sol::174 => erc20.safeApprove(address(dripsHub), type(uint256).max);
2023-01-drips\src\NFTDriver.sol::289 => erc20.safeApprove(address(dripsHub), type(uint256).max);
```

# MISSING ADDRESS(0) CHECKS

#### Impact
Issue Information: [L004]

Consider adding an address(0) check here:
#### Findings:
```
- 2023-01-drips\src\Managed.sol:: 84 => function changeAdmin(address newAdmin) public
```

# PREVENT ACCIDENTALLY BURNING TOKENS
#### Impact
Issue Information: [N001]

Transferring tokens to the zero address is usually prohibited to accidentally avoid “burning” tokens by sending them to an unrecoverable zero address.

Consider adding a check to prevent accidentally burning tokens here:


#### Findings:
```
2023-01-drips\src\AddressDriver.sol::60 => function collect(IERC20 erc20, address transferTo) 
2023-01-drips\src\DripsHub.sol::386 => function collect(uint256 userId, IERC20 erc20) 
2023-01-drips\src\NFTDriver.sol::109 => function collect(uint256 tokenId, IERC20 erc20, address transferTo)
```
