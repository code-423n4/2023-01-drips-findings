## Summary
### Low Risk Issues List
| Number |Issues Details|Count|
|:--:|:-------|:--:|
|[L-01]| Draft OpenZeppelin Dependencies | 1 |

Total 1 issues


### Non-Critical Issues List
| Number |Issues Details|Count|
|:--:|:-------|:--:|
| [N-01]|Pragma version^0.8.17  version too recent to be trusted|1|
| [N-02] | Missing NatSpec |2|
| [N-03] |Solidity compiler optimizations can be problematic| |
| [N-04] |_Lock pragmas_ to specific compiler version | 8  |
| [N-05] | Critical Address Changes Should Use Two-step Procedure |1|
| [N-06] | safeApprove of openZeppelin is deprecated| 1 |


Total 6 issues

### [L-01] Draft OpenZeppelin Dependencies

The Caller.sol contracts utilise draft-EIP712.sol, an OpenZeppelin contract. 
These contracts are still a draft and are not considered ready for mainnet use.

OpenZeppelin contracts may be considered draft contracts if they have not received adequate security auditing or are liable to change with future development.

```solidity
src/Caller.sol:

7: import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";

```

### [N-01] Pragma version ^0.8.17 version too recent to be trusted.

https://github.com/ethereum/solidity/blob/develop/Changelog.md

0.8.17 (2022-09-08)

0.8.16 (2022-08-08)

0.8.15 (2022-06-15)

0.8.10 (2021-11-09)


Unexpected bugs can be reported in recent versions;
Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs

Use a non-legacy and more battle-tested version
Use 0.8.10

### [N-02] Missing NatSpec 

```solidity
src/Splits.sol:

97:    function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal{
98:    _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
99:    }

```

```solidity
src/Caller.sol:

201:     function _call(address sender, address to, bytes memory data, uint256 value)

```

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html



### [N-03] Solidity compiler optimizations can be problematic

```js

foundry.toml:
1: [profile.default]
2: solc_version = '0.8.17'
3: optimizer_runs = 7_000
4: verbosity = 1
5: fuzz_runs = 5
6: [fmt]
7: line_length = 100
8: [fuzz]
9: runs = 4


```

**Description:**
Protocol has enabled optional compiler optimizations in Solidity.
There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. 

Therefore, it is unclear how well they are being tested and exercised.
High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. 

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported.
A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe.
It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario
A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.


**Recommendation:**
Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.
Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

### [N-04] _Lock pragmas_ to specific compiler version

**Description:**
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. 
https://swcregistry.io/docs/SWC-103

**Recommendation:**
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
[solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

```solidity
src/AddressDriver.sol:
2: pragma solidity ^0.8.17;
```

```solidity
src/Caller.sol:
2: pragma solidity ^0.8.17;
```

```solidity
src/Drips.sol:
2: pragma solidity ^0.8.17;
```

```solidity
src/DripsHub.sol:
2: pragma solidity ^0.8.17;
```

```solidity
src/ImmutableSplitsDriver.sol:
2: pragma solidity ^0.8.17;
```

```solidity
src/Managed.sol:
2: pragma solidity ^0.8.17;
```
```solidity
src/NFTDriver.sol:
2: pragma solidity ^0.8.17;
```
```solidity
src/Splits.sol:
2: pragma solidity ^0.8.17;
```


### [N-05]  Critical Address Changes Should Use Two-step Procedure

The critical procedures should be a two-step process.

Recommended Mitigation Steps

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two- step procedure on the critical functions.

```solidity
src/Managed.sol:

84:     function changeAdmin(address newAdmin) public onlyAdmin {
85:        _changeAdmin(newAdmin);
```

### [N-06] safeApprove of openZeppelin is deprecated

safeApprove from OpenZeppeling is used in the `NFTDriver.sol` contract even though its use is depreciated. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38)).

```solidity

src/NFTDriver.sol:

116:         erc20.safeTransfer(transferTo, amt);

```
