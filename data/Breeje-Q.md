## QA Report

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | USE OF `uint32` FOR TIMESTAMP | 1 |
| [L-2](#L-2) | DANGEROUS TYPE CASTING from `int128` TO `uint128` | 1 |
| [L-3](#L-3) | USE OF FLOATING PRAGMA IN FILES OUTSIDE AUTOMATED FINDINGS | 8 |
| [NC-1](#NC-1) | AVOID VARIABLE NAMES THAT CAN SHADE | 2 |

### [L-1] USE OF `uint32` FOR TIME

In entire codebase, `uint32` is used for everytime related data. It's not an issue now, But the Protocol will stop working the way it's working once it's max limit is reached in around 83 years from now.

### [L-2] DANGEROUS TYPE CASTING from `int128` TO `uint128`

The `state.amtDeltas[cycle].thisCycle` can potentially be Negative as it's `int128` type. As of now, the tests I have run are yet to fail as the `fromCycle` will most likely have positive value for `thisCycle`. But it is necessary to add a check before passing it to make sure that it is not negative given the risk associated can be very high with `receivedAmt` becoming close to max value of `uint128` because of underflow. As this `receivedAmt` is later added to spittable amount of receiver which receiver can collect so it is highly recommended to add a check.

*Instance (1) Line 289:*
```solidity
File: Drips.sol

287:      for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
288:         amtPerCycle += state.amtDeltas[cycle].thisCycle;
289:         receivedAmt += uint128(amtPerCycle);
290:         amtPerCycle += state.amtDeltas[cycle].nextCycle;
291:      }

```
[Line to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L289)

### [L-3] USE OF FLOATING PRAGMA IN FILES OUTSIDE AUTOMATED FINDINGS

Impact: [swcregistry](https://swcregistry.io/docs/SWC-103)

*Instances (8):*

All File in Scope outside the ones already found in Automated Findings uses `pragma solidity ^0.8.17;` Floating pragma Solidity Version.

### [NC-1] AVOID VARIABLE NAMES THAT CAN SHADE

*Instances (4):*
```solidity
File: Drips.sol

566:    receiver: receiver,

568:    maxEnd: maxEnd,

```
[Link to Code](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol)