## [L-01] Use of abi.encodePacked instead of bytes.concat() for Solidity version >= 0.8.4

### Description

From the Solidity version `0.8.4` it was added the possibility to use `bytes.concat` with variable number of `bytes` and `bytesNN` arguments. With a more evocative name, it functions as a restricted `abi.encodePacked`.

### Findings

- https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol => `pragma solidity ^0.8.17;`
  ```Solidity
  ::207 =>         return Address.functionCallWithValue(to, abi.encodePacked(data, sender), value);
  ```

### Resources

- [Solidity 0.8.4 Release Announcement](https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/)
- [Remove abi.encodePacked #11593](https://github.com/ethereum/solidity/issues/11593)


## [L-02] Unnamed return parameters

### Description

To increase explicitness and readability, take into account introducing and utilizing named return parameters.

### Findings

- https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol
  ```Solidity
  ::73 =>     function dripId(DripsConfig config) internal pure returns (uint32) {
  ::78 =>     function amtPerSec(DripsConfig config) internal pure returns (uint160) {
  ::83 =>     function start(DripsConfig config) internal pure returns (uint32) {
  ::88 =>     function duration(DripsConfig config) internal pure returns (uint32) {
  ::94 =>     function lt(DripsConfig config, DripsConfig otherConfig) internal pure returns (bool) {
  ```
- https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol
  ```Solidity
  ::78 =>     function admin() public view returns (address) {
  ```
- https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol
  ```Solidity
  ::294 =>     function _msgSender() internal view override(Context, ERC2771Context) returns (address) {
  ```

### References

- [Unnamed return parameters | Opyn Bull Strategy Contracts Audit](https://blog.openzeppelin.com/opyn-bull-strategy-contracts-audit/#unnamed-return-parameters)