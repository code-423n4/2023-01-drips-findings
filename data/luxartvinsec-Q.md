# 1. Integer Overflow Vulnerability in `_addSplittable` Method

Link: https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L98

Summary: 

The `_addSplittable` method in the contract can be vulnerable to an integer overflow issue. The method updates the value of splittable in the `SplitsBalance` struct by adding amt to its current value, which can cause an overflow if the `sum` of the two values exceeds the maximum value representable by a `uint128` type.

Impact: 

The integer overflow issue can result in incorrect calculation of the balance and compromise the contract's functionality. It can also lead to security issues and allow attackers to manipulate the contract's state.

Recommendation: 

To mitigate the issue, the code should be modified to check if the sum of splittable and amt exceeds the maximum value representable by a uint128 type before updating the value of splittable. If the sum exceeds the maximum value, the update should be rejected and an error should be raised.

Example:

```
function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal {
    uint128 newSplittable = _splitsStorage().splitsStates[userId].balances[assetId].splittable + amt;
    require(newSplittable >= _splitsStorage().splitsStates[userId].balances[assetId].splittable, "Overflow Error: new splittable value exceeds uint128 maximum");
    _splitsStorage().splitsStates[userId].balances[assetId].splittable = newSplittable;
}
```

# 2. Use of Uninitialized Storage Slot in NFTDriver Contract

Link : https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32

Summary: 

The NFTDriver contract uses a storage slot `_mintedTokensSlot` which is declared but never initialized. This means that it may contain arbitrary values, leading to unpredictable behavior and security risks.

Impact: 

The use of uninitialized storage slots can lead to incorrect calculation and unexpected results, potentially compromising the security of the contract and leading to financial losses for users.

Recommendation: 

The storage slot `_mintedTokensSlot` should be initialized before it is used in the contract. This can be done by calling the `StorageSlot.initializeUint256Slot` function and passing in the slot name.

Example:

```
bytes32 private immutable _mintedTokensSlot = _erc1967Slot("eip1967.nftDriver.storage");

...

constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
        ERC721("DripsHub identity", "DHI")
    {
        dripsHub = _dripsHub;
        driverId = _driverId;
        StorageSlot.initializeUint256Slot(_mintedTokensSlot, 0);
    }
```
# 3. Unsafe Minting of Tokens in NFTDriver Contract

Link : https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L62

Summary: 

The `mint` function in the NFTDriver contract is vulnerable to reentrancy attacks. An attacker could call the `mint` function and then execute a malicious contract that interacts with the NFTDriver contract, causing the attacker to receive multiple token transfers.

Impact: 

This vulnerability could allow an attacker to receive multiple token transfers, which could have significant financial impact if the tokens have value.

Recommendation: 

The `mint` function should be removed to prevent accidental use.

```
function maliciousContract(address _to, NFTDriver _nftDriver) public payable {
    uint256 tokenId = _nftDriver.mint(_to, []);
    // Do some malicious actions
    _nftDriver.mint(_to, []); // Attacker would receive two token transfers
}
```

# 4. Unchecked storage write vulnerability in `Caller` Contract

Link : https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol

Summary: 

The `Caller` contract allows an attacker to overwrite storage by writing data to the calls `mapping` in the `batch` function. This can result in unexpected behavior and data loss.

Impact: 

An attacker can overwrite data in the storage of the `Caller` contract, which can lead to unexpected behavior, data loss, and possibly other security issues.

Recommendation: 

To fix this vulnerability, a check should be added to ensure that the calls `mapping` is not being overwritten. The calls mapping should only be written to with a new key. For example:

```
require(!calls[batchId], "Batch already exists");
calls[batchId] = callArray;
```

Example: The following code demonstrates the vulnerability:

```
contract Attacker {
    function overwriteCallArray(Caller c, bytes32 batchId) public {
        uint[] memory callArray = new uint[](1);
        callArray[0] = 1;
        c.batch(batchId, callArray);
        callArray[0] = 2;
        c.batch(batchId, callArray);
    }
}
```
# 5. Reentrancy Vulnerability in Call Contract

Link : https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193

Summary:

The `Call` contract contains a reentrancy vulnerability, where an attacker can re-enter the `callBatched` function and execute arbitrary calls, potentially leading to unauthorized access or manipulation of the contract's state.

Impact: 

The vulnerability can result in loss of funds, unauthorized access or manipulation of the contract's state, or other malicious actions by attackers.

Recommendation: 

To mitigate the vulnerability, the `callBatched` function should be updated to include a mutex mechanism to prevent reentrancy.

Example: Consider the following code:

```
function callBatched(Call[] calldata calls) public {
    for (uint256 i = 0; i < calls.length; i++) {
        Call memory call = calls[i];
        address to = call.to;
        bytes memory data = call.data;
        uint256 value = call.value;
        to.call.value(value)(data);
    }
}
```

The above code does not have any protection against reentrancy, meaning an attacker can trigger a call to `callBatched` within another call to `callBatched` and execute arbitrary calls in an endless loop, potentially leading to loss of funds or manipulation of the contract's state. To prevent this, a mutex mechanism can be added, for example:

```
bool isCalling = false;
function callBatched(Call[] calldata calls) public {
    require(!isCalling, "Contract is already calling");
    isCalling = true;
    for (uint256 i = 0; i < calls.length; i++) {
        Call memory call = calls[i];
        address to = call.to;
        bytes memory data = call.data;
        uint256 value = call.value;
        to.call.value(value)(data);
    }
    isCalling = false;
}
```

# 6. Lack of Input Validation in `callBatched` Function

Link : https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193

Summary: 

The `callBatched` function in the `Caller` contract does not properly validate the inputs before executing the calls, making it susceptible to reentrancy attacks and potentially leading to loss of funds.

Impact: 

An attacker could construct a malicious contract that could steal funds from the contract or manipulate its state.

Recommendation: 

Implement proper input validation in the `callBatched` function by checking the return values of the calls before proceeding with the execution of subsequent calls.

Example:

```
function callBatched(Call[] memory calls) public {
    for (uint256 i = 0; i < calls.length; i++) {
        (bool success, ) = address(calls[i].to).call.value(calls[i].value)(calls[i].data);
        // Add check for success and revert if call fails
        require(success, "Call execution failed");
    }
}
```


