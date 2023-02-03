## 1. MISSING `address(0)` CHECKS FOR THE FOLLOWING ADDRESS INPUTS TO THE CONSTRUCTORS AND FUNCTIONS.

        erc20.safeTransfer(transferTo, amt);
	
By mistake transferTo could be set to address(0). And if `!=address(0)` check is not performed in the erc20 contract `transfer()` function the transfered tokens could be permenantly lost. 
Hence It is recommended to check for address(0) before the `erc20.safeTransfer()` is called.	

Code should be changed as follows:

	    require(transferTo != address(0), "Address cannot be zero");
        erc20.safeTransfer(transferTo, amt);
	

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L60-L63

There is 1 more instance of this issue:

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L145

## 2. REENTRANCY IN `setDrips` FUNCTION FOR ERC777 TOKENS

If the token is ERC777 (extension of ERC20) we can reenter the `setDrips` function multiple time using the hooks functionality of the ERC777. Reentrancy will be possible if the `realBalanceDelta < 0` condition is `True`, and the `TransferTo` address is a malicious contract.

        if (realBalanceDelta < 0) {
            erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
        }
		
Add openzeppelin nonReentrant modifier to `setDrips` function, to prevent re-entrancy attack using ERC777 hook functionality.
		
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L122-L147

## 3. USE `safeApprove` FUNCTION TO APPROVE TOKENS BEFORE `safeTransferFrom` FUNCTION IS CALLED.

In the `AddressDriver.sol` contract inside the `_transferFromCaller` function, `erc20.safeTransferFrom` is called as below:

        erc20.safeTransferFrom(_msgSender(), address(this), amt);
		
But the msg.sender has not approved the `AddressDriver.sol` contract to transfer tokens to its own address.
Hence it is recommended to call the `safeApprove` function by the msg.sender approving the `AddressDriver.sol` contract to transfer tokens on its behalf, before the `safeTransferFrom` is called as below:

		erc20.safeApprove(address(this), amt);
        erc20.safeTransferFrom(_msgSender(), address(this), amt);

https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L170-L171

## 4. IMMUTABLE VARIABLES SHOULD BE ASSIGNED VALUES INSIDE THE CONSTRUCTOR.

As a best practice it is recommended to assign the values of immutable variables inside the contract constructor.

       bytes32 private immutable _managedStorageSlot = _erc1967Slot("eip1967.managed.storage");	   
	   
It is recommended to assign values to the _managedStorageSlot inside constructor as follows:

       bytes32 private immutable _managedStorageSlot;
       constructor() {
           _managedStorage().isPaused = true;
		   _managedStorageSlot = _erc1967Slot("eip1967.managed.storage");
       }	   
	   
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L20

There is 1 more instance of this issue:

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L72

## 5. NO NEED TO CALL THE INHERITED CONTRACT FUNCTIONS USING THE CONTRACT'S PUBLIC API	   

The `Splits` contract is inherited by the `DripsHub` contract. Yet the `_collect()` function of the `Splits` contract is called using the contract's public API as below:

        amt = Splits._collect(userId, _assetId(erc20));
		
Since `Splits` is an inherited contract, the `_collect()` can be called by the name as below:

        amt = _collect(userId, _assetId(erc20));
		
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L392

## 6. IN THE DEFINITION OF THE DRIPS CONFIGURATION THE LAST 32 BITS ARE CONSIDERED AS DRIPS DURATION. BUT THE LOGIC IMPLEMENTATION DEFINES THE LAST 32 BITS AS THE END TIME.

In the definition given for the uint256 drips configuration the last 32 bits are considered to be the drips `duration`.
But during the logic implementation inside the `Drips.sol` contract, the last 32 bits are reffered to as the `end` timestamp of the Drips.

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L805
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L825