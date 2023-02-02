# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1] | Variables within Structs can be packed into fewer storage slots | 1 |
| [GAS-2] | Avoid compound assignment operator in state variables | 10 |
| [GAS-3] | Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead | 2 |
| [GAS-4] | 10 ** X can be changed to 1eX and save some gas | 3 |
| [GAS-5] | Using fixed bytes is cheaper than using string | 2 |
| [GAS-6] | Public functions not called by the contract should be declared external instead| 10 |
| [GAS-7] | Use Internal View Functions in Modifiers to save Bytecode| 1 |
| [GAS-8] | ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow |13 |
| [GAS-9] | Using unchecked blocks to save gas | 1 |


### [GAS-1] Variables within Structs can be packed into fewer storage slots

[Layout In Storage](https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html)

Ensure that you try to order your storage variables and struct members such that they can be packed tightly. For example, declaring your storage variables in the order of uint128, uint128, uint256 instead of uint128, uint256, uint128, as the former will only take up two slots of storage whereas the latter will take up three.

*Instances (1)*:

```diff
File: src/Drips.sol
    struct DripsState {
-       bytes32 dripsHistoryHash;
        mapping(uint256 => uint32[2 ** 32]) nextSqueezed;
        bytes32 dripsHash;
        uint32 nextReceivableCycle;
        uint32 updateTime;
        uint32 maxEnd;
        uint128 balance;
+       bytes32 dripsHistoryHash;
        uint32 currCycleConfigs;
        mapping(uint32 => AmtDelta) amtDeltas;
    }

```



### [GAS-2] Avoid compound assignment operator in state variables

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

*Instances (10)*:
```solidity
File: src/Drips.sol

253:        amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
288:        amtPerCycle += state.amtDeltas[cycle].thisCycle;
289:        receivedAmt += uint128(amtPerCycle);
290:        amtPerCycle += state.amtDeltas[cycle].nextCycle;
1053:       amtDelta.thisCycle += int128(fullCycle - nextCycle);
1054:       amtDelta.nextCycle += int128(nextCycle);
```

```solidity
File: src/DripsHub.sol

632:        totalBalances[erc20] += amt;
636:        _dripsHubStorage().totalBalances[erc20] -= amt;

```

```solidity
File: src/Splits.sol

99:         _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
168:        balance.collectable += collectableAmt;

```

### [GAS-3] Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

*Instances (2)*:

```solidity
File: src/Dripshub.sol

58:         uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;

```

```solidity
File: src/Splits.sol


22:         uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;

```


### [GAS-4] 10 ** X can be changed to 1eX and save some gas

*Instances (3)*:

```solidity
File: src/Drips.sol

191:        mapping(uint256 => uint32[2 ** 32]) nextSqueezed;
365:        uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
428:        uint32[2 ** 32] storage nextSqueezed = _dripsStorage().states[assetId][userId].nextSqueezed[senderId];
```

### [GAS-5] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

*Instances (2)*:
```solidity
File: src/Caller.sol

84:    string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("

```

```solidity
File: src/Managed.sol

136:    function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {

```

### [GAS-6] Public functions not called by the contract should be declared external instead
Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so.

*Instances (10)*:
```solidity
File: serc/AddressDriver.sol

40:    function calcUserId(address userAddr) public view returns (uint256 userId) {
60:    function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
76:    function give(uint256 receiver, IERC20 erc20, uint128 amt) public whenNotPaused {
131:   ) public whenNotPaused returns (int128 realBalanceDelta) {
158:   function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
166:   function emitUserMetadata(UserMetadata[] calldata userMetadata) public whenNotPaused {

```

```solidity
File: serc/NFTDriver.sol

50:    function nextTokenId() public view returns (uint256 tokenId) {
244:   function burn(uint256 tokenId) public override whenNotPaused {
249:   function approve(address to, uint256 tokenId) public override whenNotPaused {
272:   function setApprovalForAll(address operator, bool approved) public override whenNotPaused {
```
### [GAS-7] Use Internal View Functions in Modifiers to save Bytecode 

It is recommended to move the modifiers require statements into an internal virtual function. This reduces the size of compiled contracts that use the modifiers. Putting the require in an internal function decreases contract size when a modifier is used multiple times. There is no difference in deployment gas cost with private and internal functions.

*Instances (1)*:
```solidity
File: serc/NFTDriver.sol

40:    modifier onlyHolder(uint256 tokenId) {
41:        require(
42:            _isApprovedOrOwner(_msgSender(), tokenId),
43:            "ERC721: caller is not token owner or approved"
44:        );
45:        _;
46:    }

```

### [GAS-8] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops


*Instances (13)*:
```solidity
File: serc/Caller.sol

196:        for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: serc/Drips.sol

358:        for (uint256 i = 0; i < squeezedNum; i++) {
422:        for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
450:        for (uint256 i = 0; i < dripsHistory.length; i++) {
563:        for (uint256 i = 0; i < receivers.length; i++) {
664:        for (uint256 i = 0; i < newReceivers.length; i++) {
745:        for (uint256 i = 0; i < configsLen; i++) {
777:        for (uint256 i = 0; i < receivers.length; i++) {

```
```solidity
File: serc/DripsHub.sol

613:        for (uint256 i = 0; i < userMetadata.length; i++) {

```
```solidity
File: serc/ImmutableSplitsDriver.sol

61:        for (uint256 i = 0; i < receivers.length; i++) {

```
```solidity
File: serc/Splits.sol

126:       for (uint256 i = 0; i < currReceivers.length; i++) {
157:       for (uint256 i = 0; i < currReceivers.length; i++) {
230:       for (uint256 i = 0; i < receivers.length; i++) {

```
### [GAS-9] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block see resource
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
This will stop the check for overflow and underflow so it will save gas


*Instances (1)*:
```solidity
File: serc/Drips.sol

479:        for (uint256 idxCap = receivers.length; idx < idxCap;) {
480:            uint256 idxMid = (idx + idxCap) / 2;
481:            if (receivers[idxMid].userId < userId) {
482:                idx = idxMid + 1;
483:            } else {
484:                idxCap = idxMid;
485:            }
486:        }

```
