## Managed contracts do not have init functions despite being UUPS upgradeable 

Contracts that inherit from the Managed contract do not have initializers, even though they are meant to be called via managed proxies.
