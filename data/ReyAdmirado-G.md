
| | issue |
| ----------- | ----------- |
| 1 | [state variables should be cached in stack variables rather than re-reading them from storage](#1-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-caught-by-c4udit) |
| 2 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#2-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 3 | [not using the named return variables when a function returns, wastes deployment gas](#3-not-using-the-named-return-variables-when-a-function-returns-wastes-deployment-gas) |
| 4 | [can make the variable outside the loop to save gas](#4-can-make-the-variable-outside-the-loop-to-save-gas) |
| 5 | [`++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#5-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops) |
| 6 | [using `bool` for storage incurs overhead](#6-using-bool-for-storage-incurs-overhead) |
| 7 | [internal functions only called once can be inlined to save gas](#7-internal-functions-only-called-once-can-be-inlined-to-save-gas) |
| 8 | [usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#8-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead) |
| 9 | [Ternary over if ... else](#9-ternary-over-if--else) |
| 10 | [public functions not called by the contract should be declared external instead](#10-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 11 | [before some functions we should check some variables for possible gas save](#11-before-some-functions-we-should-check-some-variables-for-possible-gas-save) |
| 12 | [instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant](#12-instead-of-calculating-a-statevar-with-keccak256-every-time-the-contract-is-made-pre-calculate-them-before-and-only-give-the-result-to-it) |
| 13 | [Non-strict inequalities are cheaper than strict ones](#13-non-strict-inequalities-are-cheaper-than-strict-ones) |
| 14 | [Setting the constructor to payable](#14-setting-the-constructor-to-payable) |
| 15 | [instead of assigning to zero use `delete`](#15-instead-of-assigning-to-zero-use-delete) |
| 16 | [Optimize names to save gas](#16-optimize-names-to-save-gas) |



## 1. state variables should be cached in stack variables rather than re-reading them from storage (ones not caught by c4udit)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`state.updateTime` can be cached because the require check will usually be true so we save 100 gas by risking 3 more gas use.
- [Drips.sol#L542](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L542)


## 2. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas

- [Drips.sol#L253](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L253)
- [Drips.sol#L1053](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1053)
- [Drips.sol#L1054](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1054)

- [DripsHub.sol#L632](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632)

- [Splits.sol#L168](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L168)


## 3. not using the named return variables when a function returns, wastes deployment gas

- [Drips.sol#L303](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L303)
- [Drips.sol#L475](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L475)
- [Drips.sol#L538](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L538)
- [Drips.sol#L685](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L685)
- [Drips.sol#L742](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L742)
- [Drips.sol#L795](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L795)
- [Drips.sol#L818](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L818)
- [Drips.sol#L837](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L837)
- [Drips.sol#L857](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L857)
- [Drips.sol#L973](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L973)
- [Drips.sol#L991](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L991)
- [Drips.sol#L1120](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1120)
- [Drips.sol#L1128](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1128)
- [Drips.sol#L1134](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1134)

- [DripsHub.sol#L145](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L145)
- [DripsHub.sol#L160](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L160)
- [DripsHub.sol#L169](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L169)
- [DripsHub.sol#L181](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L181)
- [DripsHub.sol#L199](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L199)
- [DripsHub.sol#L317](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L317)
- [DripsHub.sol#L333](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L333)
- [DripsHub.sol#L358](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L358)
- [DripsHub.sol#L372](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L372)
- [DripsHub.sol#L465](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L465)
- [DripsHub.sol#L546](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L546)
- [DripsHub.sol#L562](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L562)
- [DripsHub.sol#L587](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L587)
- [DripsHub.sol#L598](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L598)

- [Splits.sol#L106](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L106)
- [Splits.sol#L176](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L176)
- [Splits.sol#L262](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L262)
- [Splits.sol#L273](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L273)

- [NFTDriver.sol#L50](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L50)

- [Managed.sol#L105](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L105)
- [Managed.sol#L112](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L112)
- [Managed.sol#L117](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L117)
- [Managed.sol#L136](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L136)

- [Caller.sol#L124](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L124)
- [Caller.sol#L132](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L132)
- [Caller.sol#L147](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L147)
- [Caller.sol#L171](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L171)
- [Caller.sol#L204](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L204)

- [AddressDriver.sol#L40](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L40)
- [AddressDriver.sol#L46](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L46)

- [ImmutableSplitsDriver.sol#L36](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L36)


## 4. can make the variable outside the loop to save gas

make the variable outside and only give the value to variable inside

- [Drips.sol#L361](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L361)
- [Drips.sol#L423](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L423)
- [Drips.sol#L425](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L425)
- [Drips.sol#L451](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L451)
- [Drips.sol#L452](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L452)
- [Drips.sol#L480](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L480)
- [Drips.sol#L493](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L493)
- [Drips.sol#L491](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L491)
- [Drips.sol#L564](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L564)
- [Drips.sol#L565](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L565)
- [Drips.sol#L746](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L746)
- [Drips.sol#L778](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L778)

- [DripsHub.sol#L614](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L614)

- [Splits.sol#L160](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L160)
- [Splits.sol#L163](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L163)
- [Splits.sol#L232](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L232)
- [Splits.sol#L233](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L233)
- [Splits.sol#L236](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L236)

- [Caller.sol#L197](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L197)


## 5. `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

- [Drips.sol#L247](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247)
- [Drips.sol#L287](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L287)
- [Drips.sol#L358](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L358)
- [Drips.sol#L422](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422)
- [Drips.sol#L450](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L450)
- [Drips.sol#L490](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L490)
- [Drips.sol#L563](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L563)
- [Drips.sol#L664](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L664)
- [Drips.sol#L745](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L745)
- [Drips.sol#L777](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L777)

- [DripsHub.sol#L613](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L613)

- [Splits.sol#L127](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L127)
- [Splits.sol#L158](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L158)
- [Splits.sol#L231](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L231)

- [Caller.sol#L196](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196)


## 6. using `bool` for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past

- [Managed.sol#L41](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L41)


## 7. internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

these functions can be inlined inside DripsHub.sol because its only used once there
`_receiveDrips` 
- [Drips.sol#L233](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L233)
`_receivableDripsCycles`
- [Drips.sol#L300](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L300)
`_squeezeDrips`
- [Drips.sol#L342](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L342)
`_dripsState`
- [Drips.sol#L508](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L508)
`_setDrips`
- [Drips.sol#L610](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L610)
`_splittable`
- [Splits.sol#L106](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L106)
`_splitResult`
- [Splits.sol#L117](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L117)
`_split`
- [Splits.sol#L143](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L143)
`_collectable`
- [Splits.sol#L176](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L176)
`_collect`
- [Splits.sol#L185](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L185)
`_give`
- [Splits.sol#L199](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L199)
`_setSplits`
- [Splits.sol#L212](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L212)


## 8. usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

`_AMT_PER_SEC_EXTRA_DECIMALS` 
- [Drips.sol#L108](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L108)

`_cycleSecs` (also casting is done to change it to 256 bits in some places which costs extra gas)
- [Drips.sol#L1045](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1045)


## 9. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [Drips.sol#L481-L485](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L481-L485)
- [Drips.sol#L652-L656](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L652-L656)
- [Drips.sol#L701-L704](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L701-L704)
- [Drips.sol#L709-L712](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L709-L712)
- [Drips.sol#L721-L724](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L721-L724)


## 10. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

`registerDriver`
- [DripsHub.sol#L134](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L134)
`updateDriverAddress`
- [DripsHub.sol#L152](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L152)
`nextDriverId`
- [DripsHub.sol#L160](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L160)
`cycleSecs`
- [DripsHub.sol#L169](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L169)
`receivableDripsCycles`
- [DripsHub.sol#L196](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L196)
`receiveDripsResult`
- [DripsHub.sol#L216](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L216)
`receiveDrips`
- [DripsHub.sol#L238](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L238)
`squeezeDrips`
- [DripsHub.sol#L270](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L270)
`squeezeDripsResult`
- [DripsHub.sol#L297](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L297)
`splittable`
- [DripsHub.sol#L317](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L317)
`splitResult`
- [DripsHub.sol#L328](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L328)
`split`
- [DripsHub.sol#L355](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L355)
`collectable`
- [DripsHub.sol#L372](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L372)
`collect`
- [DripsHub.sol#L386](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L386)
`dripsState`
- [DripsHub.sol#L432](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L432)
`balanceAt`
- [DripsHub.sol#L460](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L460)
`hashDrips`
- [DripsHub.sol#L546](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L546)
`hashDripsHistory`
- [DripsHub.sol#L557](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L557)
`splitsHash`
- [DripsHub.sol#L587](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L587)
`hashSplits`
- [DripsHub.sol#L595](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L595)

`mint`
- [NFTDriver.sol#L62](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L62)
`safeMint`
- [NFTDriver.sol#L79](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L79)
`collect`
- [NFTDriver.sol#L109](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L109)
`give`
- [NFTDriver.sol#L133](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L133)
`setDrips`
- [NFTDriver.sol#L186](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L186)
setSplits
- [NFTDriver.sol#L220](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L220)
`emitUserMetadata`
- [NFTDriver.sol#L235](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L235)

`changeAdmin`
- [Managed.sol#L84](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84)
`grantPauser`
- [Managed.sol#L90](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L90)
`revokePauser`
- [Managed.sol#L97](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L97)
`allPausers`
- [Managed.sol#L112](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L112)
`paused`
- [Managed.sol#L117](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L117)
`pause`
- [Managed.sol#L122](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L122)
`unpause`
- [Managed.sol#L128](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L128)

`authorize`
- [Caller.sol#L106](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L106)
`unauthorize`
- [Caller.sol#L114](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L114)
`allAuthorized`
- [Caller.sol#L132](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L132)
`callAs`
- [Caller.sol#L144](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L144)
`callSigned`
- [Caller.sol#L164](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L164)
`callBatched`
- [Caller.sol#L193](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193)


## 11. before some functions we should check some variables for possible gas save

before transfer we should check for amount being 0 with a require of if check so the function doesnt run when its not gonna do anything

- [DripsHub.sol#L394](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L394)
- [DripsHub.sol#L416](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L416)

- [NFTDriver.sol#L116](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L116)
- [NFTDriver.sol#L138](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L138)

- [AddressDriver.sol#L62](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L62)
- [AddressDriver.sol#L77](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L77)


## 12. instead of calculating a statevar with keccak256() every time the contract is made, pre calculate them before and only give the result to it

`keccak256(bytes(CALL_SIGNED_TYPE_NAME))` always has the same answer, so it can be pre calculated
- [Caller.sol#L84](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L84)

`_erc1967Slot()` function will basically do some operation on a string, all the operations can be pre calculated in these places to save gas
- [DripsHub.sol#L72](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L72)
- [NFTDriver.sol#L27](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L27)
- [Managed.sol#L20](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L20)
- [ImmutableSplitsDriver.sol#L19](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L19)


## 13. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

- [Drips.sol#L416](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L416)
- [Drips.sol#L540](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L540)
- [Drips.sol#L422](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L422)
- [Drips.sol#L748](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L748)
- [Drips.sol#L775](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L775)

- [DripsHub.sol#L631](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L631)

- [Splits.sol#L227](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L227)
- [Splits.sol#L244](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L244)

- [Caller.sol#L173](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L173)


## 14. Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

- [Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)
- [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol)
- [Splits.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol)
- [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol)
- [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol)
- [AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol)
- [ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol)


## 15. instead of assigning to zero use `delete` 

assigning to zero uses more gas than using `delete` , and they both assign variables to default value. so it is encouraged if the data is no longer needed use delete instead.

- [Splits.sol#L156](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L156)
- [Splits.sol#L188](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L188)


## 16. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas is added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signatures 

- [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol) has 33 public functions that are gonna be used externally used, ordering them for Method ID will be a huge gas save

- [NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol) has 16 public functions that are gonna be used externally used, ordering them for Method ID will be a huge gas save

- [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol) has 9 public functions that are gonna be used externally used, ordering them for Method ID will be a nice gas save

