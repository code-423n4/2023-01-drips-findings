### 1st BUG
Unsafe ERC20 Operation(s)

#### Impact
Issue Information: 
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

Example
🤦 Bad:

IERC20(token).transferFrom(msg.sender, address(this), amount);
🚀 Good (using OpenZeppelin's SafeERC20):

import {SafeERC20} from "openzeppelin/token/utils/SafeERC20.sol";

// ...

IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
🚀 Good (using require):

bool success = IERC20(token).transferFrom(msg.sender, address(this), amount);
require(success, "ERC20 transfer failed");

#### Findings:
```
2023-01-drips-main/src/NFTDriver.sol::282 => super.transferFrom(from, to, tokenId);
```
#### Tools used
Manual VS code audit

### 2nd BUG
Unspecific Compiler Version Pragma

#### Impact
Issue Information: 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

Example
🤦 Bad:

pragma solidity ^0.8.0;
🚀 Good:

pragma solidity 0.8.4;

#### Findings:
```
2023-01-drips-main/src/AddressDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips-main/src/Caller.sol::2 => pragma solidity ^0.8.17;
2023-01-drips-main/src/Drips.sol::2 => pragma solidity ^0.8.17;
2023-01-drips-main/src/DripsHub.sol::2 => pragma solidity ^0.8.17;
2023-01-drips-main/src/ImmutableSplitsDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips-main/src/Managed.sol::2 => pragma solidity ^0.8.17;
2023-01-drips-main/src/NFTDriver.sol::2 => pragma solidity ^0.8.17;
2023-01-drips-main/src/Splits.sol::2 => pragma solidity ^0.8.17;
```
#### Tools used
Manual VS code audit