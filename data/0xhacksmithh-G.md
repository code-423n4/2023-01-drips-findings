### [Gas-01] Division by 2 should always ```Bit Shifted``` and other operation which will not underflow or overflow can archive through ```Bit shift``` as well
*Instances(3)*
```solidity
File :: Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L110
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L480
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L717
```

### [Gas-02] Avoid Compound assignment operators(Specially in case of State Variables)
*Instances(18)*
```solidity
File :: Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L253
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L284
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L288-L290
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L429
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L495
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L572
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L755
```
```solidity
File :: DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L636
```
```solidity
File :: Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L128
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L159
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L162
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L167
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L168
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L235
```
```solidity
File :: ImmutableSplitsDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L62
```



### [Gas-03] ```fromCycle - maxCycles``` can be cached to memory, and used further steps in function
*Instances(1)*
```solidity
File :: Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283
```


### [Gas-04] Arithmetic Operations which will not overflow or underflow should made ```unchecked```
*Instances(19)*
```solidity
File :: Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L358
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L655
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L664
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L959
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L962
```
```solidity
File :: DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L136
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613
```
```solidity
File :: Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231
```
```solidity
File :: Caller.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196
```
```solidity
File :: ImmutableSplitsDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L59
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61
```
