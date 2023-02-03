## Managed contracts do not have init functions despite being UUPS upgradeable 

Contracts that inherit from the Managed contract do not have initializers, even though they are meant to be called via managed proxies.  Instead, they use constructors to initialize values.  However, this does not pose a security risk: the state variables that are set in the constructor are immutable, meaning they are part of the runtime code of the contract rather than the contract's storage.  

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L109-L114
