## Summary


| |Issue|Instances|
|-|:-|:-:|
| [G&#x2011;01] | Use unchecked when operation can't over/underflow | 4 |
| [G&#x2011;02] | Use private rather than public for constants | 1 |


## Gas Optimizations

### [G&#x2011;01]  Use unchecked block when operation can't over/underflow

With the version of 0.8 of Solidity, safe operations looking for over and underflow are used by default and cost extra gas. Every time that a previous condition ensure that underflow or overflow are note possible, an "unchecked" block should be add.

*There are 4 instances of this issue:*

Instances where an anterior condition makes impossible over/underflow : 

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L284


https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632


When using a for loop, you can optimize gas costs with an "unchecked" declaration for iteration.

```solidity
for (uint i; i < limit; i++) {
    //code
}

// optimized
for (uint i; i < limit; ) {
    //code
    unchecked {++i};
}
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231


### [G&#x2011;02]  Use private rather than public for constants

Even if private, the value will be in the verified contract source code. 
This way, the compiler won't create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table.

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L67

It is also the case for the other public constants of "DripsHub.sol" (lines 56,58,60,62,64 and 70), but it is less evident to
access those since the values are inherited from "Drips.sol".

