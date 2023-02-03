### [L-01] `ERC20` tokens sent to any driver may be lost
#### Impact
`ERC20` tokens sent by a user to any contract of the platform may be lost (only withdrawable after contract update).
### Proof Of Concept
ERC20 tokens are sent to the platform via the `give()` or `setDrips()` functions. However all token transfers are initiated by a `transferFrom` from the drivers and the user never has to transfer tokens by himself. A user not aware of this or mistaken could manually transfer tokens to a driver or the `DripsHub` contract, resulting in a loss of tokens.
### Recommended Mitigation Steps
Add a function to withdraw tokens stuck in the drivers and a function to withdraw tokens in `DripsHub.sol` like this:
```solidity
function withdrawStuckedTokens(ERC20 token, address to) external onlyAdmin {
	uint tokenBalance = token.balanceOf(address(this));
	uint registeredBalance = _dripsHubStorage().totalBalances[token];
	if (tokenBalance - registeredBalance != 0) {
		token.safeTransfer(to, tokenBalance - registeredBalance);
	}
}
```