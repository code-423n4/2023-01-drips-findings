## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider replacing `>=` with the strict counterpart `>` in the following inequality instance:

[File: Drips.sol#L540](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L540)

```diff
-        require(timestamp >= state.updateTime, "Timestamp before last drips update");
// Rationale for subtracting 1 on the right side of the inequality:
// x >= 10; [10, 11, 12, ...]
// x > 10 - 1 is the same as x > 9; [10, 11, 12 ...]
+        require(timestamp > state.updateTime - 1, "Timestamp before last drips update");
```
Similarly, the `<=` instance below may be replaced with `<` as follows:

[File: Drips.sol#L748](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L748)

```diff
-                if (maxEnd <= start) {
// Rationale for adding 1 on the right side of the inequality:
// x <= 10; [10, 9, 8, ...]
// x < 10 + 1 is the same as x < 11; [10, 9, 8 ...]
+                if (maxEnd < start + 1) {
```
## Unchecked SafeMath saves gas
"Checked" math, which is default in ^0.8.0 is not free. The compiler will add some overflow checks, somehow similar to those implemented by `SafeMath`. While it is reasonable to expect these checks to be less expensive than the current `SafeMath`, one should keep in mind that these checks will increase the cost of "basic math operation" that were not previously covered. This particularly concerns variable increments in for loops. When no arithmetic overflow/underflow is going to happen, `unchecked { ++i ;}` to use the previous wrapping behavior further saves gas just as in the for loop below as an example:

[File: Drips.sol#L490-L496](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490-L496)

```diff
-        for (; idx < receivers.length; idx++) {
+        for (; idx < receivers.length;) {
              DripsReceiver memory receiver = receivers[idx];

              [ ... ]

+            unchecked {
+                idx++;
+            }
          }
```
## += and -= cost more gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

For instance, the `+=` instance below may be refactored as follows:

[File: Drips.sol#L253](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L253)

```diff
-                amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
+                amtDeltas[toCycle].thisCycle = amtDeltas[toCycle].thisCycle + finalAmtPerCycle;
```
Similarly, the `-=` instance below may be refactored as follows:

[File: Drips.sol#L284](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L284)

```diff
-            toCycle -= receivableCycles;
+            toCycle = toCycle - receivableCycles;
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

[File: Drips.sol#L909](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L909)

```diff
-            if (pickCurr && pickNew) {
+            if (!(!pickCurr || !pickNew)) {
```
## Use storage instead of memory for structs/arrays
A storage pointer is cheaper since copying a state struct in memory would incur as many SLOADs and MSTOREs as there are slots. In another words, this causes all fields of the struct/array to be read from storage, incurring a Gcoldsload (2100 gas) for each field of the struct/array, and then further incurring an additional MLOAD rather than a cheap stack read. As such, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables will be much cheaper, involving only Gcoldsload for all associated field reads. Read the whole struct/array into a memory variable only when it is being returned by the function, passed into a function that requires memory, or if the array/struct is being read from another memory array/struct.

For instance, the specific example below may be refactored as follows:

[File: Drips.sol#L451](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L451)

```diff
-            DripsHistory memory drips = dripsHistory[i];
+            DripsHistory storage drips = dripsHistory[i];
```
## Ternary over `if ... else`
Using ternary operator instead of the if else statement saves gas.

For instance, the specific instance below may be refactored as follows:

[File: Drips.sol#L701-L705](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L701-L705)

```diff
      _isBalanceEnough(balance, configs, configsLen, hint1)
          ? enoughEnd = hint1
          : notEnoughEnd = hint1;
```
## Functions used only by the contract should be marked private
Functions that are only called by the contract itself should have `private` instead of `internal` visibility. This is of significant gas benefits when it is being invoked by a modifier that tends to have high usages in all other associated function calls.

For instance, `_assertCallerIsDriver()` of DripsHub.sol that is embedded in the modifier [`onlyDriver()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L118-L122) is linked to 5 other functions in the contract, i.e [`collect()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L389), [`give()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L412), [`setDrips()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L519), [`setSplits()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L579), and [`emitUserMetadata()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L611). Additionally, it is also called by [`updateDriverAddress()`](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L153).

For this reason, consider refactoring the function as follows:

[File: DripsHub.sol#L124-L126](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L124-L126)

```diff
-    function _assertCallerIsDriver(uint32 driverId) internal view {
+    function _assertCallerIsDriver(uint32 driverId) private view {
        require(driverAddress(driverId) == msg.sender, "Callable only by the driver");
    }
```
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A private visibility that saves more gas on function calls than the internal visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

[File: Managed.sol#L52-L57](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L52-L57)

```diff
+    function _onlyAdminOrPauser() private view {
+        require(
+            admin() == msg.sender || isPauser(msg.sender), "Caller is not the admin or a pauser"
+        );
+    }

    modifier onlyAdminOrPauser() {
-        require(
-            admin() == msg.sender || isPauser(msg.sender), "Caller is not the admin or a pauser"
-        );
+        _onlyAdminOrPauser();
        _;
    }
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.17",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.