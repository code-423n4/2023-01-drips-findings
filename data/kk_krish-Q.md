# Sending multiple transactions from the same account reverts with "Invalid signature".

### About the issue

This issue is common with all kinds of relayers contracts. Whenever multiple requests are sent from the same account continuously to `callSigned()` method it will result in an "Invalid signature" error. The cause for this behavior is that the state of the nonce(`nonce[sender]++`) will not get updated. This makes the function be called only synchronously, one after another. 
But in reality, there will be a lot of use cases for a single account to send multiple transactions at the same time. 

Line: https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L181

### Recommended mitigation steps
One way to circumvent this issue is to use 2D nonces. Biconomy(gasless txns relayer) is using this technique to mitigate this issue.
Using the 2D nonces technique, the sender can send their own nonce to separate from another set of transactions.
Reference code from Biconomy: https://github.com/bcnmy/mexa/blob/master/contracts/6/forwarder/BiconomyForwarder.sol#L35