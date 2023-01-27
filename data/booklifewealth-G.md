https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61&#L63

replace state variable reads within loops with local variable reads . This is done by assigning state variable values to new local variables, reading  the local variables in a loop, then after the loop assigning any changed local variables to their equivalent state variables.

Here is the optimized code:

assign the following state variables to local variables
receivers.length
receivers[i].weight;  


The optimised code takes the state reads  out of the loop so that they are done one time instead of multiple times.