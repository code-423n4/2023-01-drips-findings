## Non-critical issues
## [N-01] Use a more recent version of solidity

For security, it is best practice to use the latest Solidity version.

An old version is used (0.8.17), it is recommended to use the latest version (0.8.18)

*There are 8 instances of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L2#Lpragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L2#Lpragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L2#Lpragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L2#Lpragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L2#Lpragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L2#Lpragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L2#Lpragma solidity ^0.8.17;
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L2#Lpragma solidity ^0.8.17;

---
## Low risk issues
## [L-01] Use Two-Step Transfer Pattern for Access Controls  

Contracts implementing access control's, e.g. owner, should consider implementing a Two-Step Transfer pattern.

Otherwise it's possible that the role mistakenly transfers ownership to the wrong address, resulting in a loss of the role.

*There is 1 instance of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84-L86
