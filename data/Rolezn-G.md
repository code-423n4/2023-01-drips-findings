## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 4 | 400 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Setting the `constructor` to `payable` | 7 | 91 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Using fixed bytes is cheaper than using `string` | 1 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Using `delete` statement can save gas | 11 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Use hardcoded address instead `address(this)` | 6 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | `internal` functions only called once can be inlined to save gas | 23 | 506 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate | 2 | - |
| [GAS&#x2011;8](#GAS&#x2011;8) | Optimize names to save gas | 8 | 176 |
| [GAS&#x2011;9](#GAS&#x2011;9) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 21 | - |
| [GAS&#x2011;10](#GAS&#x2011;10) | Public Functions To External | 42 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | Help The Optimizer By Saving A Storage Variable’s Reference Instead Of Repeatedly Fetching It | 15 | - |
| [GAS&#x2011;12](#GAS&#x2011;12) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 53 | - |
| [GAS&#x2011;13](#GAS&#x2011;13) | `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops | 15 | 525 |
| [GAS&#x2011;14](#GAS&#x2011;14) | Using `unchecked` blocks to save gas | 1 | 136 |
| [GAS&#x2011;15](#GAS&#x2011;15) | Using `storage` instead of `memory` saves gas | 10 | 3500 |
| [GAS&#x2011;16](#GAS&#x2011;16) | Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states | 3 | 15000 |

Total: 222 contexts over 16 issues

## Gas Optimizations

### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
176: abi.encode(
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L176

```solidity
842: return keccak256(abi.encode(receivers));
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L842

```solidity
858: return keccak256(abi.encode(oldDripsHistoryHash, dripsHash, updateTime, maxEnd));
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L858

```solidity
278: return keccak256(abi.encode(receivers));
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L278








### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
30: constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L30

```solidity
219: constructor(uint32 cycleSecs, bytes32 dripsStorageSlot)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L219

```solidity
109: constructor(uint32 cycleSecs_)
        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
        Splits(_erc1967Slot("eip1967.splits.storage"))
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L109

```solidity
28: constructor(DripsHub _dripsHub, uint32 _driverId)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L28

```solidity
158: constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0))
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L158

```solidity
32: constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
        ERC2771Context(forwarder)
        ERC721("DripsHub identity", "DHI")
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L32

```solidity
94: constructor(bytes32 splitsStorageSlot)
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L94





### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Using fixed bytes is cheaper than using `string`

As a rule of thumb, use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

#### <ins>Proof Of Concept</ins>


```solidity
82: string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L82






### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
478: uint256 idx = 0;
489: uint256 amt = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L478

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L489



```solidity
744: uint256 spent = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L744

```solidity
880: uint256 currIdx = 0;
881: uint256 newIdx = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L880-L881

```solidity
60: uint256 weightSum = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L60

```solidity
126: uint32 splitsWeight = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L126

```solidity
156: balance.splittable = 0;
157: uint32 splitsWeight = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L156-L157

```solidity
188: balance.collectable = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L188

```solidity
228: uint64 totalWeight = 0;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L228






### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
171: erc20.safeTransferFrom(_msgSender(), address(this), amt);
173: if (erc20.allowance(address(this), address(dripsHub)) == 0) {

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L171

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L173



```solidity
416: erc20.safeTransferFrom(msg.sender, address(this), amt);

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L416

```solidity
533: erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L533

```solidity
286: erc20.safeTransferFrom(_msgSender(), address(this), amt);
288: if (erc20.allowance(address(this), address(dripsHub)) == 0) {

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L286

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L288





#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`






### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>

```solidity
73: function dripId

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L73

```solidity
83: function start

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L83

```solidity
88: function duration

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L88

```solidity
233: function _receiveDrips
270: function _receiveDrips

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L233

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L270



```solidity
342: function _squeezeDrips
392: function _squeezeDrips

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L342

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L392



```solidity
610: function _setDrips

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L610

```solidity
136: function _erc1967Slot

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L136

```solidity
151: function _authorizeUpgrade

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L151

```solidity
300: function _msgData

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L300

```solidity
106: function _splittable

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L106

```solidity
106: function _split
117: function _split
143: function _split
262: function _split
283: function _split

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L106

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L117

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L143

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L262

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L283



```solidity
176: function _collectable

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L176

```solidity
176: function _collect
185: function _collect

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L176

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L185



```solidity
199: function _give

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L199

```solidity
212: function _setSplits

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L212

```solidity
262: function _splitsHash

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L262






### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

#### <ins>Proof Of Concept</ins>


```solidity
88: mapping(address => EnumerableSet.AddressSet) internal _authorized;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L88

```solidity
90: mapping(address => uint256) public nonce;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L90






### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
253: amtDeltas[toCycle].thisCycle += finalAmtPerCycle;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L253

```solidity
284: toCycle -= receivableCycles;
288: amtPerCycle += state.amtDeltas[cycle].thisCycle;
289: receivedAmt += uint128(amtPerCycle);
290: amtPerCycle += state.amtDeltas[cycle].nextCycle;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L284

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L288

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L289

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L290



```solidity
429: amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L429

```solidity
495: amt += _drippedAmt(receiver.config.amtPerSec(), start, end);

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L495

```solidity
572: balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L572

```solidity
755: spent += _drippedAmt(amtPerSec, start, end);

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L755

```solidity
1053: amtDelta.thisCycle += int128(fullCycle - nextCycle);
1054: amtDelta.nextCycle += int128(nextCycle);

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1053-L1054

```solidity
632: totalBalances[erc20] += amt;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L632

```solidity
636: _dripsHubStorage().totalBalances[erc20] -= amt;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L636

```solidity
62: weightSum += receivers[i].weight;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L62

```solidity
99: _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L99

```solidity
128: splitsWeight += currReceivers[i].weight;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L128

```solidity
159: splitsWeight += currReceivers[i].weight;
162: splitAmt += currSplitAmt;
167: collectableAmt -= splitAmt;
168: balance.collectable += collectableAmt;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L159

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L162

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L167

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L168



```solidity
235: totalWeight += weight;

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L235






### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function calcUserId(address userAddr) public view returns (uint256 userId) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L40

```solidity
function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L60

```solidity
function give(uint256 receiver, IERC20 erc20, uint128 amt) public whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L76

```solidity
function setDrips(
        IERC20 erc20,
        DripsReceiver[] calldata currReceivers,
        int128 balanceDelta,
        DripsReceiver[] calldata newReceivers,
        
        uint32 maxEndHint1,
        uint32 maxEndHint2,
        address transferTo
    ) public whenNotPaused returns (int128 realBalanceDelta) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L122

```solidity
function setSplits(SplitsReceiver[] calldata receivers) public whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L158

```solidity
function emitUserMetadata(UserMetadata[] calldata userMetadata) public whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L166

```solidity
function authorize(address user) public {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L106

```solidity
function unauthorize(address user) public {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L114

```solidity
function isAuthorized(address sender, address user) public view returns (bool authorized) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L124

```solidity
function allAuthorized(address sender) public view returns (address[] memory authorized) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L132

```solidity
function callSigned(
        address sender,
        address to,
        bytes memory data,
        uint256 deadline,
        bytes32 r,
        bytes32 sv
    ) public payable returns (bytes memory returnData) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L164

```solidity
function callBatched(Call[] memory calls) public payable returns (bytes[] memory returnData) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L193

```solidity
function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L134

```solidity
function driverAddress(uint32 driverId) public view returns (address driverAddr) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L145

```solidity
function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L152

```solidity
function nextDriverId() public view returns (uint32 driverId) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L160

```solidity
function cycleSecs() public view returns (uint32 cycleSecs_) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L169

```solidity
function totalBalance(IERC20 erc20) public view returns (uint256 balance) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L181

```solidity
function squeezeDrips(
        uint256 userId,
        IERC20 erc20,
        uint256 senderId,
        bytes32 historyHash,
        DripsHistory[] memory dripsHistory
    ) public whenNotPaused returns (uint128 amt) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L270

```solidity
function squeezeDripsResult(
        uint256 userId,
        IERC20 erc20,
        uint256 senderId,
        bytes32 historyHash,
        DripsHistory[] memory dripsHistory
    ) public view returns (uint128 amt) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L297

```solidity
function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L317

```solidity
function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L372

```solidity
function balanceAt(
        uint256 userId,
        IERC20 erc20,
        DripsReceiver[] memory receivers,
        uint32 timestamp
    ) public view returns (uint128 balance) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L460

```solidity
function setDrips(
        uint256 userId,
        IERC20 erc20,
        DripsReceiver[] memory currReceivers,
        int128 balanceDelta,
        DripsReceiver[] memory newReceivers,
        
        uint32 maxEndHint1,
        uint32 maxEndHint2
    ) public whenNotPaused onlyDriver(userId) returns (int128 realBalanceDelta) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L510

```solidity
function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L546

```solidity
function hashDripsHistory(
        bytes32 oldDripsHistoryHash,
        bytes32 dripsHash,
        uint32 updateTime,
        uint32 maxEnd
    ) public pure returns (bytes32 dripsHistoryHash) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L557

```solidity
function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L587

```solidity
function nextUserId() public view returns (uint256 userId) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L36

```solidity
function admin() public view returns (address) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L78

```solidity
function changeAdmin(address newAdmin) public onlyAdmin {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L84

```solidity
function grantPauser(address pauser) public onlyAdmin {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L90

```solidity
function revokePauser(address pauser) public onlyAdmin {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L97

```solidity
function isPauser(address pauser) public view returns (bool isAddrPauser) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L105

```solidity
function allPausers() public view returns (address[] memory pausersList) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L112

```solidity
function paused() public view returns (bool isPaused) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L117

```solidity
function pause() public onlyAdminOrPauser whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L122

```solidity
function unpause() public onlyAdminOrPauser whenPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L128

```solidity
function nextTokenId() public view returns (uint256 tokenId) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L50

```solidity
function setDrips(
        uint256 tokenId,
        IERC20 erc20,
        DripsReceiver[] calldata currReceivers,
        int128 balanceDelta,
        DripsReceiver[] calldata newReceivers,
        
        uint32 maxEndHint1,
        uint32 maxEndHint2,
        address transferTo
    ) public whenNotPaused onlyHolder(tokenId) returns (int128 realBalanceDelta) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L186

```solidity
function burn(uint256 tokenId) public override whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L244

```solidity
function approve(address to, uint256 tokenId) public override whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L249

```solidity
function setApprovalForAll(address operator, bool approved) public override whenNotPaused {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L272







### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>


```solidity
244: DripsState storage state = _dripsStorage().states[assetId][userId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L244

```solidity
248: delete amtDeltas[cycle];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L248

```solidity
253: amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L253

```solidity
286: DripsState storage state = _dripsStorage().states[assetId][userId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L286

```solidity
288: amtPerCycle += state.amtDeltas[cycle].thisCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L288

```solidity
290: amtPerCycle += state.amtDeltas[cycle].nextCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L290

```solidity
356: DripsState storage state = _dripsStorage().states[assetId][userId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L356

```solidity
357: uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L357

```solidity
410: DripsState storage sender = _dripsStorage().states[assetId][senderId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L410

```solidity
420: _dripsStorage().states[assetId][userId].nextSqueezed[senderId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L420

```solidity
481: if (receivers[idxMid].userId < userId) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L481

```solidity
491: DripsReceiver memory receiver = receivers[idx];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L491

```solidity
620: DripsState storage state = _dripsStorage().states[assetId][userId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L620

```solidity
639: _dripsStorage().states[assetId],
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L639

```solidity
149: SplitsBalance storage balance = splitsStates[userId].balances[assetId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L149






### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
25: uint32 public immutable driverId;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/AddressDriver.sol#L25

```solidity
191: uint32 updateTime;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L191

```solidity
193: uint32 maxEnd;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L193

```solidity
108: uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L108

```solidity
117: uint32 internal immutable _cycleSecs;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L117

```solidity
189: uint32 nextReceivableCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L189

```solidity
195: uint128 balance;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L195

```solidity
197: uint32 currCycleConfigs;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L197

```solidity
208: int128 thisCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L208

```solidity
210: int128 nextCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L210

```solidity
237: uint32 receivableCycles;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L237

```solidity
238: uint32 fromCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L238

```solidity
239: uint32 toCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L239

```solidity
240: int128 finalAmtPerCycle;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L240

```solidity
357: uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L357

```solidity
365: uint32 cycleStart = _currCycleStart();
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L365

```solidity
421: uint32 squeezeEndCap = _currTimestamp();
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L421

```solidity
425: uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L425

```solidity
487: uint32 updateTime = dripsHistory.updateTime;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L487

```solidity
488: uint32 maxEnd = dripsHistory.maxEnd;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L488

```solidity
623: uint32 lastUpdate = state.updateTime;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L623

```solidity
624: uint128 newBalance;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L624

```solidity
625: uint32 newMaxEnd;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L625

```solidity
627: uint32 currMaxEnd = state.maxEnd;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L627

```solidity
923: uint32 currStartCycle = _cycleOf(currStart);
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L923

```solidity
924: uint32 newStartCycle = _cycleOf(newStart);
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L924

```solidity
949: uint32 startCycle = _cycleOf(start);
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L949

```solidity
997: uint40 end = uint40(start) + receiver.config.duration();
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L997

```solidity
1135: uint32 currTimestamp = _currTimestamp();
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L1135

```solidity
58: uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L58

```solidity
64: uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L64

```solidity
97: uint32 nextDriverId;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L97

```solidity
119: uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L119

```solidity
15: uint32 public immutable driverId;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L15

```solidity
17: uint32 public immutable totalSplitsWeight;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L17

```solidity
25: uint32 public immutable driverId;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/NFTDriver.sol#L25

```solidity
11: uint32 weight;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L11

```solidity
22: uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L22

```solidity
88: uint128 splittable;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L88

```solidity
90: uint128 collectable;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L90

```solidity
157: uint32 splitsWeight = 0;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L157

```solidity
161: uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L161

```solidity
228: uint64 totalWeight = 0;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L228

```solidity
233: uint32 weight = receiver.weight;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L233






### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

#### <ins>Proof Of Concept</ins>


```solidity
196: for (uint256 i = 0; i < calls.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L196

```solidity
247: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L247

```solidity
287: for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L287

```solidity
358: for (uint256 i = 0; i < squeezedNum; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L358

```solidity
422: for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L422

```solidity
450: for (uint256 i = 0; i < dripsHistory.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L450

```solidity
479: for (uint256 idxCap = receivers.length; idx < idxCap;) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L479

```solidity
490: for (; idx < receivers.length; idx++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L490

```solidity
563: for (uint256 i = 0; i < receivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L563

```solidity
664: for (uint256 i = 0; i < newReceivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L664

```solidity
613: for (uint256 i = 0; i < userMetadata.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/DripsHub.sol#L613

```solidity
61: for (uint256 i = 0; i < receivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/ImmutableSplitsDriver.sol#L61

```solidity
127: for (uint256 i = 0; i < currReceivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L127

```solidity
158: for (uint256 i = 0; i < currReceivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L158

```solidity
231: for (uint256 i = 0; i < receivers.length; i++) {
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L231





### <a href="#Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isnâ€™t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

```solidity
780: require(_isOrdered(receivers[i - 1], receiver), "Receivers not sorted");

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L780





### <a href="#Summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Using `storage` instead of `memory` saves gas

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another `memory` array/struct

#### <ins>Proof Of Concept</ins>


```solidity
197: Call memory call = calls[i];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Caller.sol#L197

```solidity
423: DripsHistory memory drips = dripsHistory[dripsHistory.length - i];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L423

```solidity
451: DripsHistory memory drips = dripsHistory[i];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L451

```solidity
491: DripsReceiver memory receiver = receivers[idx];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L491

```solidity
564: DripsReceiver memory receiver = receivers[i];
778: DripsReceiver memory receiver = receivers[i];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L564

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L778



```solidity
665: DripsReceiver memory receiver = newReceivers[i];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L665

```solidity
564: DripsReceiver memory receiver = receivers[i];
778: DripsReceiver memory receiver = receivers[i];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L564

https://github.com/code-423n4/2023-01-drips/tree/main/src/Drips.sol#L778



```solidity
232: SplitsReceiver memory receiver = receivers[i];

```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Splits.sol#L232





### <a href="#Summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from `true` to `false` can cost up to ~20000 gas rather than `uint256(2)` to `uint256(1)` that would cost significantly less.

#### <ins>Proof Of Concept</ins>

```solidity
    struct ManagedStorage {
        bool isPaused;
        EnumerableSet.AddressSet pausers;
    }
```
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L40-L43


```solidity
74: _managedStorage().isPaused = true;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L74

```solidity
123: _managedStorage().isPaused = true;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L123

```solidity
129: _managedStorage().isPaused = false;
```

https://github.com/code-423n4/2023-01-drips/tree/main/src/Managed.sol#L129






