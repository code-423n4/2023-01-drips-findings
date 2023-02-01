ERC20 operations pose a risk due to variations in implementation and vulnerabilities in the standard. To mitigate these issues, consider using either the OpenZeppelin SafeERC20 library or enclosing each operation in a require statement. Also, ERC20's approve functions are vulnerable to race-condition attacks, so it's recommended to use OpenZeppelin's SafeERC20 library's safeIncrease or safeDecrease Allowance functions to address this.

Lines

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L250
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L282