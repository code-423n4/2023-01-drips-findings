## QA Report Summary:
1. Low-Risk Findings:
L-1: Critical Address change should be done in 2 steps.
L-2: Lack of Address(0) checks.
L-3: Lock the Pragma or decrease it
L-4: Unbounded Loops
L-5: Centralization Risk & No use of Timelocks:

2. Non-Critical Risk Findings:
NC-1: Wrong Function Ordering.
NC-2: Large multiples of ten should use scientific notation.
NC-3: OpenZeppelin Draft Contract usage, Directly Import EIP712.sol.
NC-4: Events missing for critical changes.
NC-5: Use a single file for all system-wide constants.
NC-6: Define contracts in separate files:
NC-7: Use `encode` instead of `encodePacked`

3. Informational/Suggestions:
I-1: Typo.
I-2: Test Coverage.
I-3: Improve the code quality.


## Low-Risk Findings:
### L-1. Critical Address change should be done in 2 steps.
In Managed.sol, a [function](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84) allows an admin to change it to a different address. If the admin accidentally uses an invalid address for which they do not have the private key, then the system gets locked.
It is important to have two steps for admin change where the first is announcing a pending new admin and the new address should then claim its ownership.
```
    function changeAdmin(address newAdmin) public onlyAdmin {  // @audit low - lack of 2 steps approach
        _changeAdmin(newAdmin);
    }
```


### L-2. Lack of Address(0) checks.
There are 7 instances of this issue:
Must add a address(0) check here: [Managed.sol:84](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84)
```
src / Managed.sol {Managed}:
84: changeAdmin(address newAdmin)
```
Consider adding a address(0) check here especially for Immutable addresses:
[AddressDriver.sol:33](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L33), 
[Caller.sol:106](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L106), 
[ImmutableSplitsDriver:28](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L28),
[Managed.sol:Managed:90](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L90),
[Managed.sol:ManagedProxy:158](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L158), 
[NFTDriver.sol:32](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L32)


### L-3. Lock the Pragma or decrease it
Contracts should be deployed with the same compiler version, that they have been tested thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, a compiler version that might introduce bugs that affect the contract system negatively. - [SWC-103](https://swcregistry.io/docs/SWC-103).

Also, if you want floating pragma so other projects can integrate Drips contracts, then must consider dropping the pragma. ^0.8.17 is a very recent version.  I would recommend using at least 0.8.13 in that case. As most projects use 0.8.7-0.8.13 solidity compiler versions, which are considered more stable compiler versions. Or Simply lock the pragma, `0.8.17`.


### L-4. Unbounded Loops
Some functions have unbounded loops, which can cause DoS for the users. Consider setting an upper limit.
[Caller.sol:196](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L196):
```
for (uint256 i = 0; i < calls.length; i++){
```
[ImmutableSplitsDriver.sol:61](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L61):
```
for (uint256 i = 0; i < receivers.length; i++){
```

### L-5. Centralization Risk & No use of Timelocks:
`admin` role in the project: Admin is not behind a multi-sig (not much info is given about this in the docs or natspec comments).
Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions like `_authorizeUpgrade()`. In such a case, users who have invested in the project will suffer high financial losses.

Also, the `changeAdmin` [function](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84) is not behind a timelock

Consider, Adding a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks & Allow only multi-signature wallets to call the function to reduce the likelihood of an attack. Also, detail them in the documentation and NatSpec comments.


## Non-Critical Risk Findings:

### NC-1. Wrong Function Ordering.
Project contracts don't follow the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#), especially in function ordering. Many `view` functions were defined before public/external/internal functions & some private functions were defined in between public/external functions. It was confusing & was making the audit process a little difficult. Pls consider using this order in your contracts for better readability:
```
constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
Within a grouping, place the view and pure functions last.
```
The wrong function ordering was made in the following contracts:
[AddressDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol), 
[Caller.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol),
[Drips.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol),
[ImmutableSplitsDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol),
[NFTDriver.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol),
[Splits.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol).

### NC-2. Large multiples of ten should use scientific notation (e.g. 1e6 ) rather than decimal literals (e.g. 1000000 ), for readability.
There are 2 instances of this issue:
[Drips.sol#L110](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L110):
```
uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
```
[Splits.sol#L22](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L22):
```
uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
```

### NC-3. OpenZeppelin Draft Contract usage, Directly Import EIP712.sol.
[Caller.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L5) is importing `draft-EIP712.sol`, which is a [drafted contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/draft-EIP712.sol). EIP-712 is Finalised. This file is deprecated. Consider directly importing [EIP712.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/EIP712.sol.

### NC-4. Events missing for critical changes.
Events help non-contract tools to track changes, and events prevent users from being surprised by changes,
Emit Events here: [Managed.sol:84](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84) & here: [NFTDriver.sol:79](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L79)


### NC-5. Use a single file for all system-wide constants.
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access those values). This will help with readability and easier maintenance for future changes. 
```
Caller.sol:
    string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned(""address sender,address to,bytes data,uint256 value,uint256 nonce,uint256 deadline)";

Drips.sol:
    uint256 internal constant _MAX_DRIPS_RECEIVERS = 100;
    uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
    uint256 internal constant _AMT_PER_SEC_MULTIPLIER = 1_000_000_000;
    uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);

DripsHub.sol:
    uint256 public constant MAX_DRIPS_RECEIVERS = _MAX_DRIPS_RECEIVERS;
    uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
    uint256 public constant AMT_PER_SEC_MULTIPLIER = _AMT_PER_SEC_MULTIPLIER;
    uint256 public constant MAX_SPLITS_RECEIVERS = _MAX_SPLITS_RECEIVERS;
    uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
    uint256 public constant DRIVER_ID_OFFSET = 224;
    uint256 public constant MAX_TOTAL_BALANCE = _MAX_TOTAL_DRIPS_BALANCE;

Splits.sol:
    uint256 internal constant _MAX_SPLITS_RECEIVERS = 200;
    uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
    uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;
```

### NC-6. Define contracts in separate files:
In [Managed.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol), there are two contracts defined, [Managed](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L18) & [ManagedProxy](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L157). 
Consider separating them for better understanding and readability.

### NC-7. Use `encode` instead of `encodePacked`
Use of abi.encodePacked is safe, but unnecessary and not recommended. abi.encodePacked can result in hash collisions when used with two dynamic arguments (string/bytes).

There is also a discussion of removing abi.encodePacked from future versions of Solidity (Ethereum/solidity#11593), so using abi.encode now will ensure compatibility in the future as well.

Affected source code:
[Caller.sol:207](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L207):
```
 return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);
```

## Informational/Suggestions:
### I-1. Typo.
There is a typo in [splits.t.sol](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/test/Splits.t.sol#L5). Not in just the contest repo, In the project's actual [repo](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/test/Splits.t.sol#L5) as well. Because of this Splits.sol was not working as well.
Change `import {Splits, SplitsReceiver} from "src//Splits.sol";` --> `import {Splits, SplitsReceiver} from "src/Splits.sol";`

### I-2. Test Coverage.
The test coverage for drips contracts is around 80%-90%. It is essential to create comprehensive tests with 100% coverage for smart contracts to ensure proper functionality & security.

### I-3. Improve the code quality.
For better code quality, firstly use [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#). Also, consider using [code headers](https://github.com/transmissions11/headers) like [ArtGobblers](https://github.com/artgobblers/art-gobblers/blob/a18ea7fec3b766444a3277392c405e2a107a2d75/src/ArtGobblers.sol#L277) contracts. 
```
    /*//////////////////////////////////////////////////////////////
                               CONSTRUCTOR
    //////////////////////////////////////////////////////////////*/
```