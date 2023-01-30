# QA report

## Low Risk

| L-R    |Issue|
|:------:|:----|
| [L&#x2011;01] | Missing zero address check | 9 |
| [L&#x2011;02] | _safeMint is vulnerable to reentrancy | 1 |
| [L&#x2011;03] | Use of deprecated safeApprove of openZeppelin's SafeERC20 | 2 |
| [L&#x2011;04] | Use of one step ownership transfer in `Managed.sol` | 2 |

Total: 4 issues

## Non-critical

| N-C    |Issue|
|:------:|:----|
| [NC&#x2011;01] | Do not use floating pragma, use a concrete version instead  | 2 |
| [NC&#x2011;02] | Constants should be defined rather than using magic numbers | 6 |
| [NC&#x2011;03] | 1_000_000_000 can be changed to 1e8 for readability | 11 |
| [NC&#x2011;04] | Event is missing indexed fields| 19 |
| [NC&#x2011;05] | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,) | 31 |

Total: 5 issues

## Low Risk

### [L-01] Missing zero address check

File: `DripsHub.sol`

If in `registerDriver()`, `driverAddr` is accidentally set to 0 address, then `updateDriverAddress()` will be impossible to call because it requires to be called only by a valid driver address.  

File: `DripsHub.sol`

```solidity
function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
..
function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
 _assertCallerIsDriver(driverId);
```

Lines of code: 

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L134

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152

Consider adding zero address check.

### [L-02]  _safeMint is vulnerable to reentrancy

This function is safe because it checks whether the receiver can receive ERC721 tokens. The can prevent the case that a NFT will be minted to a contract that cannot handle ERC721 tokens. However, this external function call creates a security loophole. Specifically, the attacker can perform a reentrant call inside the onERC721Received callback. More detailed information why a reeentrancy can occur - https://blocksecteam.medium.com/when-safemint-becomes-unsafe-lessons-from-the-hypebears-security-incident-2965209bda2a.

File: `NFTDriver.sol`

```solidity
_safeMint(to, tokenId);
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L85


Use the nonReentrant modifier from ReentrancyGuard contract by OpenZeppelin in the functions where you use _safeMint.

### [L-03] Use of deprecated safeApprove of openZeppelin's SafeERC20
The use of safeApprove() is deprecated as described in comments
Use of safeIncreaseAllowance() and safeDecreaseAllowance() is recommended.
It appears in following codebase:

File: `AddressDriver.sol`

```solidity
erc20.safeApprove(address(dripsHub), type(uint256).max);
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L174

### [L-04] Use of one step ownership transfer in `Managed.sol`

It is a best practice to use two-step ownership transfer pattern, meaning ownership transfer gets to a "pending" state and the new owner should claim his new rights, otherwise the old owner still has control of the contract. Consider using OpenZeppelin's Ownable2Step contract.

```solidity
/// Can only be called by the current admin.
function changeAdmin(address newAdmin) public onlyAdmin {
_changeAdmin(newAdmin);
    }
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84


## Non-critical

### [NC-01] Do not use floating pragma, use a concrete version instead 
Using a floating pragma statement is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode. The problem persist in all of the contracts.

```solidity
pragma solidity ^0.8.17;
```

### [NC-02] Constants should be defined rather than using magic numbers

File: `Drips.sol`

```solidity
uint256 end = (enoughEnd + notEnoughEnd) / 2;
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L717

### [NC-03] 1_000_000_000 can be changed to 1e8 for readability 

File: `Drips.sol`

```solidity
uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
```

Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L110


### [NC-04] Event is missing indexed fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

File: `Managed.sol`

```solidity
event PauserGranted(address indexed pauser, address admin);

event PauserRevoked(address indexed pauser, address admin);

event Paused(address pauser);

event Unpaused(address pauser);
```
Lines of code:

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L25

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L30

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L34

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L38

### [NC-05] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).

File: `Caller.sol`

```solidity
return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);
```

Lines of code: 

- https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L207