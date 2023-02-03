## [G-01] Use assembly to hash instead of Solidity

Example :
```
function solidityHash(uint256 a, uint256 b) public view {
        //unoptimized
        keccak256(abi.encodePacked(a, b));
    }

function assemblyHash(uint256 a, uint256 b) public view {
        //optimized
        assembly {
            mstore(0x00, a)
            mstore(0x20, b)
            let hashedVal := keccak256(0x00, 0x40)
        }
    }
```
*There are 7 instances of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L137
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L842
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L858
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L278
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L84
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L175
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L177

---
## [G-02] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. Overflow/underflow can be checked in assembly to ensure safety.

Example :
```
    //addition in Solidity
    function addTest(uint256 a, uint256 b) public pure {
        uint256 c = a + b;
    }

    //addition in assembly
    function addAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := add(a, b)

            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }
```

*There are 28 instances of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L137
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L282
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L283
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L306
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L361
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L362
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L363
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L366
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L423
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L425
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L480
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L482
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L636
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L717
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L780
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L806
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L997
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1045
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1047
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1053
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1122
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1137
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L631
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L37
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L130
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L131
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L161
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L51

---

## [G-03] Tightly pack storage variables

When defining storage variables, make sure to declare them in ascending order, according to size. When multiple variables are able to fit into one 256 bit slot, this will save storage size and gas during runtime. For example, if you have a bool, uint256 and a bool, instead of defining the variables in the previously mentioned order, defining the two boolean variables first will pack them both into one storage slot since they only take up one byte of storage.

*There are 2 instances of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L103-L119
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L53-L72

---

## [G-04] Mark storage variables as constant if they never change.

State variables can be declared as constant or immutable. In both cases, the variables cannot be modified after the contract has been constructed. For constant variables, the value has to be fixed at compile-time, while for immutable, it can still be assigned at construction time.

The compiler does not reserve a storage slot for these variables, and every occurrence is inlined by the respective value.

Compared to regular state variables, the gas costs of constant and immutable variables are much lower. For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. This allows for local optimizations. Immutable variables are evaluated once at construction time and their value is copied to all the places in the code where they are accessed. For these values, 32 bytes are reserved, even if they would fit in fewer bytes. Due to this, constant values can sometimes be cheaper than immutable values.

*There are 5 instances of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L20
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L72
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L19
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L27
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L84

---

## [G-05] Optimal Comparison

When comparing integers, it is cheaper to use strict > & < operators over >= & <= operators, even if you must increment or decrement one of the operands. Note: before using this technique, it's important to consider whether incrementing/decrementing one of the operators could result in an over/underflow. This optimization is applicable when the optimizer is turned off.

*There are 9 instances of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L416
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L540
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L748
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L775
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L631
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L227
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L244
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L173

---
## [G-06] Setting the constructor to payable 

Making the constructor to payable can eliminate the check for `msg.value`, saving 13 gas on deployment

*There are 8 instances of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L73
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L158
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L94