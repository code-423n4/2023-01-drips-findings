1. Remove the return statement in the constructor. It is unnecessary. 
   - DripsHub.sol
   ```
    constructor(uint32 cycleSecs_)
        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
        Splits(_erc1967Slot("eip1967.splits.storage"))
    {
        return;
    }
   ``` 
   - [Link](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L109-L114)