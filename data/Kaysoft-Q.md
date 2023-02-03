## [L-01] AVOID FLOATING PRAGMA


Files: All files
see: https://swcregistry.io/docs/SWC-103


#### Recommended Mitigation Steps
Consider using `pragma solidity 0.8.17;` instead of `pragma solidity ^0.8.17;`

## [L-02] `_authorizeUpgrade` function not implemented

The `_authorizeUpgrade` function not implemented
File:
- [src/Managed.sol#L151](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L151)

```
function _authorizeUpgrade(address /* newImplementation */ ) internal view override onlyAdmin {
        return;
    }
```


## [NC-01] ADD NatSpec comments to functions

Files: 

- [src/NFTDriver.sol#L285](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L285)

-[src/NFTDriver.sol#L300](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L300)
