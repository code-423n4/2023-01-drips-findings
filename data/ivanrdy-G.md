<h4>Gas optimization through calldata</h4>

<h6>For array parameters in functions, it is recommended to use `calldata` over `memory` , as this provides significant gas savings.<h6>

```
 function allPausers() public view returns (address[] memory pausersList) {
        return _managedStorage().pausers.values();
    }
```

```
function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
        internal
        view
        returns (uint128 collectableAmt, uint128 splitAmt)
    {
```

```
 function allAuthorized(address sender) public view returns (address[] memory authorized) {
        return _authorized[sender].values();
    }
```

```
 function _split(uint256 userId, uint256 assetId, SplitsReceiver[] memory currReceivers)
        internal
        returns (uint128 collectableAmt, uint128 splitAmt)
    {
```

<h5>Affected souce code:</h5>
<h6>https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Managed.sol/112</h6>
<h6>https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol/117</h6>
<h6>https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Caller.sol/132</h6>
<h6>https://github.com/code-423n4/2023-01-drips/blob/0d35c60b8a482528e8347d576bb9aef6fa7964cc/src/Splits.sol/143</h6>

