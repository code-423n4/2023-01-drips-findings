## Managed contracts do not have init functions despite being UUPS upgradeable 

### Impact

Contracts that inherit from the Managed contract and the Managed contract, itself, do not have initializers.  This will lead to storage collisions in managed proxies of DripsHub since the ERC1967 storage slots of Splits and Drips are set in their respective constructors, which are not callable by proxies.  Therefore, the ERC1967 storage slots for Drips and Splits will be equal to 0x00.  

### Remediation

Any contract meant to be called via proxy should use initializers with a factory contract.  