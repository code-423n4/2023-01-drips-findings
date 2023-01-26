
Lack of zero-address validation on address parameters 
it may lead to transaction reverts, waste gas, require resubmission
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L144 166 202
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L152 
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84 97 158
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32 62 79 254 263 277 

lacks the keyword payable
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L170 76 122

transfer zero amount can be reverted
Some tokens don't allow transfer of 0 amount.
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L170 76
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L286

abi.encode() is less efficient than abi.encodePacked()
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L176
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L842 858
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L278