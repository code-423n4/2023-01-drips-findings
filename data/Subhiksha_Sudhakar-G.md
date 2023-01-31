### Finding 1:  A += B costs more gas than A = A + B

[Drips.sol#L253](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L253)
[Drips.sol#L288](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L288)
[Drips.sol#L289](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L289)
[Drips.sol#L290](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L290)
[Drips.sol#L429](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L429)
[Drips.sol#L495](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L495)
[Drips.sol#L755](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L755)
[Drips.sol#L1053](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1053)
[Drips.sol#L1054](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1054)

[DripsHub.sol#L632](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632)

[ImmutableSplitsDriver.sol#L62](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L62)

[Splits.sol#L128](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L128)
[Splits.sol#L159](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L159)
[Splits.sol#L162](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L162)
[Splits.sol#L168](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L168)
[Splits.sol#L235](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L235)


---


### Finding 2:  Public functions can be set as external 

Public and external functions differs in terms of gas usage. The former use more gass than the latter. This is due to the fact that Solidity copies arguments to memory on a public function while external read from calldata which is cheaper than memory allocation.

The following public functions could be set external to save gas and improve code quality.

[Caller.sol#L106](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L106)
[Caller.sol#L114](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L114)
[Caller.sol#L132](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L132)
[Caller.sol#L144](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L144)
[Caller.sol#L193](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193)
[Caller.sol#L144](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L144)

[DripsHub.sol#L134](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L134)
[DripsHub.sol#L152](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L152)
[DripsHub.sol#L160](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L160)
[DripsHub.sol#L169](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L169)
[DripsHub.sol#L181](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L181)
[DripsHub.sol#L196](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L196)
[DripsHub.sol#L216](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L216)
[DripsHub.sol#L238](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L238)
[DripsHub.sol#L270](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L270)
[DripsHub.sol#L297](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L297)
[DripsHub.sol#L317](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L317)
[DripsHub.sol#L328](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L328)
[DripsHub.sol#L355](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L355)
[DripsHub.sol#L372](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L372)
[DripsHub.sol#L386](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L386)
[DripsHub.sol#L409](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L409)
[DripsHub.sol#L432](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L432)
[DripsHub.sol#L460](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L460)
[DripsHub.sol#L519](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L519)
[DripsHub.sol#L546](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L546)
[DripsHub.sol#L562](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L562)
[DripsHub.sol#L576](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L576)
[DripsHub.sol#L595](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L595)
[DripsHub.sol#L608](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L608)

[ImmutableSplitsDriver.sol#L53](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L53)

[Managed.sol#L90](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L90)
[Managed.sol#L97](https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L97)

[NFTDriver.sol#L62](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L62)
[NFTDriver.sol#L79](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L79)
[NFTDriver.sol#L109](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L109)
[NFTDriver.sol#L133](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L133)
[NFTDriver.sol#L196](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L196)
[NFTDriver.sol#L220](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L220)

