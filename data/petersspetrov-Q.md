[QA-01]Better code readability
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L216-L219
		
		
## Proof of Concept	
if (newSplitsHash != state.splitsHash) {
            _assertSplitsValid(receivers, newSplitsHash);
            state.splitsHash = newSplitsHash;
        }
		
## Recommended Mitigation Steps:
		if (newSplitsHash == state.splitsHash) return;
        _assertSplitsValid(receivers, newSplitsHash);
        state.splitsHash = newSplitsHash;
	
	
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L173-L175	

## Proof of Concept	
if (erc20.allowance(address(this), address(dripsHub)) == 0) {
            erc20.safeApprove(address(dripsHub), type(uint256).max);
        }	
		
## Recommended Mitigation Steps:
		(erc20.allowance(address(this), address(dripsHub)) != 0) return;
		{
            erc20.safeApprove(address(dripsHub), type(uint256).max);
        }


[L-01]The Emit to be after the action 
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L65		
File: ImmutableSplitsDriver.sol
## Proof of Concept	

    emit CreatedSplits(userId, dripsHub.hashSplits(receivers));
    dripsHub.setSplits(userId, receivers);		
    if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);
		
		
## Recommended Mitigation Steps:

    dripsHub.setSplits(userId, receivers);		
    if (userMetadata.length > 0) dripsHub.emitUserMetadata(userId, userMetadata);
	emit CreatedSplits(userId, dripsHub.hashSplits(receivers));
	