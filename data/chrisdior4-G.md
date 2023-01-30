# Gas Optimization report

| G-O    |Issue|
|:------:|:----|
| [N&#x2011;01] | Division by two should use bit shifting | 2 |
| [N&#x2011;02] | x = x - y costs less gas than x -= y, same for addition | 6 |
| [N&#x2011;03] | THERE’S NO NEED TO SET DEFAULT VALUES FOR VARIABLES | 11 |
| [N&#x2011;04] | Use custom errors instead of revert strings | 19 |
| [N&#x2011;05] | Cache Array Length Outside of Loop | 31 |
| [N&#x2011;06] | ++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too) | 2 |
| [N&#x2011;07] | REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS | 1 |
| [N&#x2011;08] | BYTES CONSTANTS ARE MORE EFFICIENT THAN STRING CONSTANTS | 1 |
| [N&#x2011;09] | Use x != 0 instead of x > 0 for uint types | 1 |
| [N&#x2011;10] | ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW | 1 |

Total: 10 issues



### [G-01]  Division by two should use bit shifting

<x>/ 2 is the same as <x>>\> 1\. While the compiler uses the SHR opcode to accomplish both, the version that uses division incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two</x></x>

File: `Drips.sol`

```solidity
uint256 end = (enoughEnd + notEnoughEnd) / 2;
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L717


### [G-02] x = x - y costs less gas than x -= y, same for addition
You can replace all -= and += occurrences to save gas

Example:

File: `Drips.sol`

```solidity
amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L253


### [G-03] THERE’S NO NEED TO SET DEFAULT VALUES FOR VARIABLES
If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

File: `Drips.sol`

```solidity
uint256 idx = 0;
```
```solidity
uint256 amt = 0;
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L478

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L489


### [G-04] Use custom errors instead of revert strings
Solidity 0.8.4 added the custom errors functionality, which can be use instead of revert strings, resulting in big gas savings on errors. Replace all revert statements with custom error ones.

Example:

File: `Drips.sol`

```solidity
require(historyHash == finalHistoryHash, "Invalid drips history");
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L461


### [G-05] Cache Array Length Outside of Loop
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

File: `Drips.sol`

```solidity
 for (; idx < receivers.length; idx++) {
```

Consider changing it to:

```solidity
uint256 receiversLength = receivers.length;
for (; idx < receiversLength; idx++) {
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L490

There are a other instances of this issue, change every one to save gas.

### [G-06]  ++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)

File: `Drips.sol`

```solidity
for (uint256 i = 0; i < squeezedNum; i++) {
```
Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L358

There are a other instances of this issue, change every one to save gas.



### [G-07] REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS
Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas.

File: `Drips.sol`

```solidity
require(dripsHash == 0, "Drips history entry with hash and receivers");
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L454

File: `NFTDriver.sol`

```solidity
require(_isApprovedOrOwner(_msgSender(), tokenId),"ERC721: caller is not token owner or approved"
   );
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L43


### [G-08] BYTES CONSTANTS ARE MORE EFFICIENT THAN STRING CONSTANTS
If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.  

File: `Caller.sol`

```solidity
string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("
```

Lines of code:

-  https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L82

### [G-09] Use x != 0 instead of x > 0 for uint types
The != operator costs less gas than > and for uint types you can use it to check for non-zero values to save gas

File: `DripsHub.sol`

```solidity
if (receivedAmt > 0) {
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L245

### [G-10] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR-LOOP AND WHILE-LOOPS
In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

File: `Caller.sol`

```solidity
for (uint256 i = 0; i < calls.length; i++) {
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L196

This is just one example, the problem persists in couple of places in the contracts.
