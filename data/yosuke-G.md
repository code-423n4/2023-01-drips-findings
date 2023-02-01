## [YO GAS-1] Use calldata instead of memory for function arguments that do not get mutated.

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L202
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L347
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L397
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L446
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L472
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L536
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L559
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L613
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L615
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L682
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L739
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L769
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L792
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L815
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L874
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L877
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L970
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L986
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1061
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L136
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L117
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L143
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L212
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L226


### Recommended Mitigation Steps
Use calldata instead of memory for function arguments that do not get mutated because of lower gas fee.
ex)
before

```solidity=
function _call(address sender, address to, bytes memory data, uint256 value)
```

after

```solidity=
function _call(address sender, address to, bytes calldata data, uint256 value)
```

## [YO GAS-2] USE CUSTOM ERRORS RATHER THAN REVERT()/REQUIRE() STRINGS TO SAVE GAS

### Handle
yosuke

## Vulnerability details
### Impact
Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas.

### Proof of Concept
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L53
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L254
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L41


### Recommended Mitigation Steps
USE CUSTOM ERRORS RATHER THAN REVERT()/REQUIRE() STRINGS TO SAVE GAS

## [YO GAS-3] Using private rather than public for constants and immutable, saves gas

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L23
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L25
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L13-L17
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L23-L25

### Recommended Mitigation Steps
ex)
before

```solidity=
DripsHub public immutable dripsHub;
```

after

```solidity=
DripsHub private immutable dripsHub;
```

## [YO GAS-4] USING BOOLS FOR STORAGE INCURS OVERHEAD

### Handle
yosuke

## Vulnerability details
### Impact
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and  pointer aliasing, and it cannot be disabled.

### Proof of Concept
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L124
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L883
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L890
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1064
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L41
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L105
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L117


### Recommended Mitigation Steps
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

## [YO GAS-5] `++I`/`I++` SHOULD BE `UNCHECKED{++I}`/`UNCHECKED{I++}` WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN `FOR`AND `WHILE`LOOPS

### Handle
yosuke

## Vulnerability details
### Impact

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

### Proof of Concept
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L358
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L664
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L745
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L777
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231


### Recommended Mitigation Steps

```solidity
for (uint256 i; i < length;) {
	...
	unchecked{++i}
}
```

## [YO GAS-6]X = X + Y, X = X - Y, X = X * Y, X = X / Y ARE MORE EFFICIENT, THAN X += Y, X -= Y, X *= Y, X /= Y

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L253
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L284
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L288-L290
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L429
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L495
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L572
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L755
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1053
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1054
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L636
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L62
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L128
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L159
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L162
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L167
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L168
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L235

### Recommended Mitigation Steps
ex)
before

```solidity=
splitsWeight += currReceivers[i].weight;
```

after

```solidity=
splitsWeight = splitsWeight + currReceivers[i].weight;
```

## [YO GAS-7] USE FUNCTION INSTEAD OF MODIFIERS

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L46
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L52
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L60
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L66
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L40

### Recommended Mitigation Steps
USE FUNCTION INSTEAD OF MODIFIERS because of lower gas fee.
ex)
before

```solidity=
modifier onlyAdmin() {
    require(admin() == msg.sender, "Caller is not the admin");
    _;
}
```

after

```solidity=
function onlyAdmin() private {
    require(admin() == msg.sender, "Caller is not the admin");
}
```