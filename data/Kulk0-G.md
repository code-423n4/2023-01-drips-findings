1. In Managed.sol onlyAdminOrPauser modifier the require string has over 32 bytes. If you shorten it you will save a lot of gas on deploy. https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L54

2. In Managed.sol onlyAdmin modifier you call `admin()` function in the require statement that in turn calls `_getAdmin()`. This is unnecessary, if you switch admin() call to _getAdmin() call you will save gas since it won't have to jump through the bytecode so much. https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L47

3. In Managed.sol onlyAdminOrPauser() is the same thing. If you switch admin() with _getAdmin() you will save gas https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L54:

```solidity
modifier onlyAdmin() {
    // Here call _getAdmin() instead of admin()
    require(admin() == msg.sender, "Caller is not the admin");
    _;
}
```

4. In drips.sol there are 11 different for loops, if you moved the increment mechanism like `cycle++` to unchecked block the user will save around 80 gas per for loop cycle. So if a user is collecting rewards for many cycles he could potentially save thousands of units of gas.

from:
```solidity
for (uint256 i = 0; i < receivers.length; i++) {
    // do stuff
}
```
to:
```solidity
for (uint256 i = 0; i < receiers.length;) {
    // do stuff
    unchecked {
        i++
    }
}
```
5. In Caller.sol the same thing. If you switch the for loop you can save gas.

6. In DripsHub.sol the same thing. If you switch the for loop you can save a lot of gas

7. In Splits.sol there are 3 for loops so the same thing.

