
###### https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

The function ***callBatched() line 196*** can revert if calls.length is too long, there should be a limit of 5 calls gas to avoid that there is a revert of the function.

Still function ***callBatched() line 196***, the variable I does not need to be initialized to 0.
