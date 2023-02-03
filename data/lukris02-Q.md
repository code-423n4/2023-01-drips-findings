# QA Report for Drips Protocol contest
## Overview

During the audit, 5 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
NC-1 | Order of Functions | Non-Critical | 7
NC-2 | Order of Layout | Non-Critical | 6
NC-3 | Missing NatSpec | Non-Critical | 5
NC-4 | Missing leading underscore | Non-Critical | 1
NC-5 | Unused named return variables | Non-Critical | 44

## Non-Critical Risk Findings(5)
### NC-1. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
Private functions between internal:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L314
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L555

Internal functions between private:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L508-L543
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L834-L859

Internal function betfore public:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L124

Internal functions between public:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L91
- https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L46

##### Recommendation
Reorder functions where possible.
#
### NC-2. Order of Layout
##### Description
According to [Order of Layout](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout), inside each contract, library or interface, use the following order:
1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions
##### Instances
Structs should be placed before events:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L173-L211
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L95
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L73-L91
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L40

Modifiers should be placed before constructor:
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L118
- https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L40

##### Recommendation
Place modifiers before constructor, and place structs before events.
#
### NC-3. Missing NatSpec
##### Description
NatSpec is missing for 5 functions in 4 contracts.
##### Instances
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L629
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L635
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L98
- https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L285
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L202

##### Recommendation
Add NatSpec for all functions.
#
### NC-4. Missing leading underscore
##### Description
Internal functions should have a leading underscore.
##### Instances
- https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L46

#
### NC-5. Unused named return variables
##### Description
Both named return variable(s) and return statement are used.
##### Instances
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L306
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L497
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L520
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L542 
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L806
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L825
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L858
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L975
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1012
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1122
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1129
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1137 
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L146 
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L161
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L170
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L182
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L201
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L318
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L333
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L360
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L373
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L443
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L466
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L547
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L563
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L588
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L600
- https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L643
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L107
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L177
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L263
- https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L51 
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L106 
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L113
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L118
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L137
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L125
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L133
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L150
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L182
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L207
- https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L41
- https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L47
- https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L37

##### Recommendation
To improve clarity use only named return variables.  
For example, change:
```
function functionName() returns (uint id) {
    return x;
```
to
```
function functionName() returns (uint id) {
    id = x;
```