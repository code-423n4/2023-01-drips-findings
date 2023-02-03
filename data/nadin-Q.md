## QA Issues List
Issue 
## [N-01] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
## [N-02] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
## [N-03] PRAGMA VERSION^0.8.17 VERSION TOO RECENT TO BE TRUSTED.
## [N-04] No use of upgradeable OpenZeppelin contract
## [N-05] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE
## [N-06] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
##  [L-01] DRAFT OPENZEPPELIN DEPENDENCIES 
Total: 38 contexts over 07 issues

## [N-01] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
Context: 08 All Contracts
Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
Recommendation:
NatSpec comments should be increased in contracts

## [N-02] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103
Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/
Contexts: 08

    2: pragma solidity ^0.8.17;

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol

## [N-03] PRAGMA VERSION^0.8.17 VERSION TOO RECENT TO BE TRUSTED.
Contexts: All Contracts
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

## [N-04] No use of upgradeable OpenZeppelin contract
Cotexts: 10
File: main/src/DripsHub.sol
Make use of Open Zeppelins https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol

    import {SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L8

File: main/src/NFTDriver.sol
Make use of Open Zeppelins https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/metatx/ERC2771ContextUpgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/StorageSlotUpgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC721/extensions/ERC721BurnableUpgradeable.sol

    import {Context, ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
    import {IERC20, SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";
    import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";
    import {
        ERC721,
        ERC721Burnable,
        IERC721
    } from "openzeppelin-contracts/token/ERC721/extensions/ERC721Burnable.sol";

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L6-L13

File: main/src/Managed.sol
Make use of Open Zeppelins https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/StorageSlotUpgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/structs/EnumerableSetUpgradeable.sol

    import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";
    import {EnumerableSet} from "openzeppelin-contracts/utils/structs/EnumerableSet.sol";

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L6-L7

File: main/src/Caller.sol
Make use of Open Zeppelins https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/structs/EnumerableSetUpgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/metatx/ERC2771ContextUpgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/AddressUpgradeable.sol,

    import {Address} from "openzeppelin-contracts/utils/Address.sol";
    import {ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
    import {EnumerableSet} from "openzeppelin-contracts/utils/structs/EnumerableSet.sol";

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L4-L7

File: main/src/AddressDriver.sol
Make use of Open Zeppelins https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/metatx/ERC2771ContextUpgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-

    import {ERC2771Context} from "openzeppelin-contracts/metatx/ERC2771Context.sol";
    import {SafeERC20} from "openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L13-L14

File: main/src/ImmutableSplitsDriver.sol
Make use of Open Zeppelins  https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/StorageSlotUpgradeable.sol, https://github.com/OpenZeppelin/openzeppelin-contracts-

    import {StorageSlot} from "openzeppelin-contracts/utils/StorageSlot.sol";

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L6

## [N-05] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE
Contexts: 02
File: main/src/Managed.sol

    84:    function changeAdmin(address newAdmin) public onlyAdmin {
    85:        _changeAdmin(newAdmin);
    86:    }

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L84-L86
Recommended Mitigation Steps
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two- step procedure on the critical functions.

## [N-06] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).
Contexts: 01
File: main/src/Caller.sol

    207:        return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L207

##  [L-01] DRAFT OPENZEPPELIN DEPENDENCIES 
Contexts: 01
File: main/src/Caller.sol

    import {ECDSA, EIP712} from "openzeppelin-contracts/utils/cryptography/draft-EIP712.sol";

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L5