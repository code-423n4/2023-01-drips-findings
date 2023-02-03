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