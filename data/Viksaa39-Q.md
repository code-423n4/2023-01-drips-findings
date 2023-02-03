[L] - Usage of draft-EIP712.sol in Caller.sol. recommend using EIP712.sol over the existing draft version

[NC] - Unused imports in

1. Managed.sol - Line 6
2. AddressDriver.sol - Line 5 , only DripsHistory
3. DripsHub.sol - Line 4, DripsConfig and DripsConfigImpl
4. NFTDriver.sol - Line 4, only DripsHistory

[NC] - emitUserMetadata function in AddressDriver.sol(L166) and NFTDriver.sol(L235) is not being used anywhere. No reason why such a function should exist.

[L] - Reentrancy is possible if ERC777 is used as a token. But not sure if any funds can be stolen.

In Line 174 of AddressDriver.sol - We can see a _transferFromCaller() which has a safeTransferFrom. This function will have a callback if ERC777 is used. So an attacker can implement a tokensToSend function in theri contract and can call any function BEFORE safeTransferFrom executes.

This _transferFromCaller() is used by many other functions,

Another reentrancy possibility can be seen in safeTransfer function which is used in functions like collect and setDrips. in this case, an attacker can do the same as above but the code in the attacker contract will be executed AFTER the transfer function execute