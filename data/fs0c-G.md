## [G-01] In the splits functionality an extra variable declaration can be reduced.

Current Implementation:

```solidity
function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)
        internal
        returns (uint128 collectableAmt, uint128 splitAmt)
    {
        _assertCurrSplits(userId, currReceivers);
        mapping(uint256 => SplitsState) storage splitsStates = _splitsStorage().splitsStates;
        SplitsBalance storage balance = splitsStates[userId].balances[assetId];
```

Recommended:

```solidity
function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)
        internal
        returns (uint128 collectableAmt, uint128 splitAmt)
    {
        _assertCurrSplits(userId, currReceivers);
        SplitsBalance storage balance = _splitsStorage().splitsStates[userId].balances[assetId];
```

Gas usage before in calling `testCollectTransfersFundsToTheProvidedAddress:NFTDriver.t.sol` :207571 

Gas usage after in calling `testCollectTransfersFundsToTheProvidedAddress:NFTDriver.t.sol` :207567