### GAS Optimization for the Caller contract

**Severity: Low**

**Status: Vulnerable**

****File(s) affected:****  [https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol)

**Description:** 

The review of the `Caller` contract showed that arguments with complex data types are passed with a keyword `memory` instead of `calldata` what would make the calls relatively cheaper in terms of GAS. Especially it make sense to implement the protocol as efficient as possible, because I assume the GAS fees for Meta Transactions will be paid and covered by project’s accounts.

The function arguments with `memory` keyword can be changed to `calldata`, because those arguments are just passed forward to the call and are not modified in any line of execution: 

```
1. function callAs(address sender, address to, bytes memory data) public payable returns (bytes memory returnData)
2. function callSigned(
        address sender,
        address to,
        bytes memory data,
        uint256 deadline,
        bytes32 r,
        bytes32 sv
    ) public payable returns (bytes memory returnData)
3. function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData)
4. function _call(address sender, address to, bytes memory data, uint256 value) internal returns (bytes memory returnData) {
```

In addition, all the `public` functions of the Caller contract can be changed to `external` visibility, it will also save GAS fees. 

**Recommendation:**

Switch the memory keyword to calldata for the functions mentioned in the **Description** section, and change visibility for all `public` function to `external`.