
# QA Report

## [L-01] use a safer ownership transfer method

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L84](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L84)

### Impact

The protocol may lose ownership, though the possibility is relatively low. 

### Recommendation

`changeAdmin` should be 2-step structure admin ownership transfer to make sure it is done correctly. 

This is one of the examples of using the 2-step ownership transfer. And if the protocol is not supposed to renounce the ownership, there should be a check to see the address is not a zero address. 

## [L-02] do not use a floating solidity version

All contracts in the repo used this: 

```solidity
pragma solidity ^0.8.17;
```

It is recommended as the best practice to use a fixed version of solidity. 

### Recommendation

Rewrite it as below: 

```solidity
pragma solidity 0.8.17;
```

## [N-01] delete the unnecessary import

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L6](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/Managed.sol#L6)

The StorageSlot is not used in the contract. So it is not necessary to import the contract. 

## [N-02] create a modifier for simplicity

[https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L124-L126](https://github.com/radicle-dev/drips-contracts/blob/aa149127572e467af4c1adf4c2a37160b39bc91b/src/DripsHub.sol#L124-L126)

Create a modifier for this function for more simplicity of the code. 

I have noticed that the function was used in several places in the code base. So it would be better if we can have a modifier of it. 