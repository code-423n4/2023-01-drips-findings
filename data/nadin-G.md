## Gas Optimizations
Issue 
## [G-01] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
## [G-02] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
## [G-03] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE
## [G-04] SETTING THE CONSTRUCTOR TO PAYABLE
## [G-05] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS:
## [G-06] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
## [G-07] ADDED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT
## [G-08] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
## [G-09] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT
## [G-10] Functions guaranteed to revert when called by normal users can be marked payable
## [G‑11] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS
## [G-12] USE BYTES32 INSTEAD OF STRING
Total: 343 contexts over 12 issues

## [G-01] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Contexts: 212
File: main/src/Drips.sol

    28:    uint32 updateTime;
    30:    uint32 maxEnd;
    60:    function create(uint32 dripId_, uint160 amtPerSec_, uint32 start_, uint32 duration_)
    74:        return uint32(DripsConfig.unwrap(config) >> 224);
    79:        return uint160(DripsConfig.unwrap(config) >> 64);
    84:        return uint32(DripsConfig.unwrap(config) >> 32);
    89:        return uint32(DripsConfig.unwrap(config));
    108:    uint8 internal constant _AMT_PER_SEC_EXTRA_DECIMALS = 9;
    112:    uint256 internal constant _MAX_TOTAL_DRIPS_BALANCE = uint128(type(int128).max);
    117:    uint32 internal immutable _cycleSecs;
    133:        bytes32 indexed receiversHash,
    134:        bytes32 dripsHistoryHash,
    135:        uint128 balance,
    136:        uint32 maxEnd
    153:        uint256 indexed userId, uint256 indexed assetId, uint128 amt, uint32 receivableCycles
    169:        uint128 amt,
    185:        mapping(uint256 => uint32[2 ** 32]) nextSqueezed;
    189:        uint32 nextReceivableCycle;
    191:        uint32 updateTime;
    193:        uint32 maxEnd;
    195:        uint128 balance;
    197:        uint32 currCycleConfigs;
    203:        mapping(uint32 => AmtDelta) amtDeltas;
    208:        int128 thisCycle;
    210:        int128 nextCycle;
    219:    constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
    233:    function _receiveDrips(uint256 userId, uint256 assetId, uint32 maxCycles)
    235:        returns (uint128 receivedAmt)
    237:        uint32 receivableCycles;
    238:        uint32 fromCycle;
    239:        uint32 toCycle;
    240:        int128 finalAmtPerCycle;
    246:            mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
    247:            for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
    270:    function _receiveDripsResult(uint256 userId, uint256 assetId, uint32 maxCycles)
    274:            uint128 receivedAmt,
    275:            uint32 receivableCycles,
    276:            uint32 fromCycle,
    277:            uint32 toCycle,
    278:            int128 amtPerCycle
    287:        for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
    289:            receivedAmt += uint128(amtPerCycle);
    303:        returns (uint32 cycles)
    305:        (uint32 fromCycle, uint32 toCycle) = _receivableDripsCyclesRange(userId, assetId);
    317:        returns (uint32 fromCycle, uint32 toCycle)
    348:    ) internal returns (uint128 amt) {
    357:        uint32[2 ** 32] storage nextSqueezed = state.nextSqueezed[senderId];
    402:            uint128 amt,
    419:        uint32[2 ** 32] storage nextSqueezed =
    421:        uint32 squeezeEndCap = _currTimestamp();
    425:                uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];
    473:        uint32 squeezeStartCap,
    474:        uint32 squeezeEndCap
    475:    ) private view returns (uint128 squeezedAmt) {
    487:        uint32 updateTime = dripsHistory.updateTime;
    488:        uint32 maxEnd = dripsHistory.maxEnd;
    493:            (uint32 start, uint32 end) =
    497:        return uint128(amt);
    514:            uint32 updateTime,
    515:            uint128 balance,
    516:            uint32 maxEnd
    527:        uint32 timestamp
    556:        uint128 lastBalance,
    557:        uint32 lastUpdate,
    558:        uint32 maxEnd,
    560:        uint32 timestamp
    565:            (uint32 start, uint32 end) = _dripsRange({
    572:            balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
    614:        int128 balanceDelta,
    617:        uint32 maxEndHint1,
    618:        uint32 maxEndHint2
    619:    ) internal returns (int128 realBalanceDelta) {
    623:        uint32 lastUpdate = state.updateTime;
    624:        uint128 newBalance;
    625:        uint32 newMaxEnd;
    627:            uint32 currMaxEnd = state.maxEnd;
    628:            int128 currBalance = int128(
    636:            newBalance = uint128(currBalance + realBalanceDelta);
    681:        uint128 balance,
    683:        uint32 hint1,
    684:        uint32 hint2
    685:    ) private view returns (uint32 maxEnd) {
    692:                return uint32(enoughEnd);
    695:            uint256 notEnoughEnd = type(uint32).max;
    697:                return uint32(notEnoughEnd);
    855:        uint32 updateTime,
    856:        uint32 maxEnd
    875:        uint32 lastUpdate,
    876:        uint32 currMaxEnd,
    878:        uint32 newMaxEnd
    912:                (uint32 currStart, uint32 currEnd) =
    914:                (uint32 newStart, uint32 newEnd) =
    923:                uint32 currStartCycle = _cycleOf(currStart);
    924:                uint32 newStartCycle = _cycleOf(newStart);
    936:                (uint32 start, uint32 end) = _dripsRangeInFuture(currRecv, lastUpdate, currMaxEnd);
    944:                (uint32 start, uint32 end) =
    949:                uint32 startCycle = _cycleOf(start);
    970:    function _dripsRangeInFuture(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd)
    973:        returns (uint32 start, uint32 end)
    975:        return _dripsRange(receiver, updateTime, maxEnd, _currTimestamp(), type(uint32).max);
    987:        uint32 updateTime,
    988:        uint32 maxEnd,
    989:        uint32 startCap,
    990:        uint32 endCap
    991:    ) private pure returns (uint32 start, uint32 end_) {
    997:        uint40 end = uint40(start) + receiver.config.duration();
    1012:        return (start, uint32(end));
    1020:    function _addDeltaRange(DripsState storage state, uint32 start, uint32 end, int256 amtPerSec)
    1027:        mapping(uint32 => AmtDelta) storage amtDeltas = state.amtDeltas;
    1037:        mapping(uint32 => AmtDelta) storage amtDeltas,
    1048:            AmtDelta storage amtDelta = amtDeltas[_cycleOf(uint32(timestamp))];
    1053:            amtDelta.thisCycle += int128(fullCycle - nextCycle);
    1054:            amtDelta.nextCycle += int128(nextCycle);
    1120:    function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
    1128:    function _currTimestamp() private view returns (uint32 timestamp) {
    1129:        return uint32(block.timestamp);
    1134:    function _currCycleStart() private view returns (uint32 timestamp) {
    1135:        uint32 currTimestamp = _currTimestamp();

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L60
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L28
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L30
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L60
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L74
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L79
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L84
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L89
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L108
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L112
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L117
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L133
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L134
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L135
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L136
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L153
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L169
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L185
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L189
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L191
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L193
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L195
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L197
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L203
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L208
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L210
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L219
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L233
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L235
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L237
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L238
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L239
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L240
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L246
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L247
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L270
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L274
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L275
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L276
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L277
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L278
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L287
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L289
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L303
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L305
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L317
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L348
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L357
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L402
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L419
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L421
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L425
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L473
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L474
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L475
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L487
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L488
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L493
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L497
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L514
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L515
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L516
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L537
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L556
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L557
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L558
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L560
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L565
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L572
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L614
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L617
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L618
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L619
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L623
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L624
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L625
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L627
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L628
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L636
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L681
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L683
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L684
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L685
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L692
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L695
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L697
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L719
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L855
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L856
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L875
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L876
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L878
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L912
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L914
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L923
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L924
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L936
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L944
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L949
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L970
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L973
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L975
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L987
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L988
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L989
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L990
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L991
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L997
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1012
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1020
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1027
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1037
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1048
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1053
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1054
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1120
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1128
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1129
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1134
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1135

File: main/src/DripsHub.sol

    58:    uint8 public constant AMT_PER_SEC_EXTRA_DECIMALS = _AMT_PER_SEC_EXTRA_DECIMALS;
    64:    uint32 public constant TOTAL_SPLITS_WEIGHT = _TOTAL_SPLITS_WEIGHT;
    77:    event DriverRegistered(uint32 indexed driverId, address indexed driverAddr);
    84:        uint32 indexed driverId, address indexed oldDriverAddr, address indexed newDriverAddr
    97:        uint32 nextDriverId;
    99:        mapping(uint32 => address) driverAddresses;
    109:    constructor(uint32 cycleSecs_)
    119:        uint32 driverId = uint32(userId >> DRIVER_ID_OFFSET);
    124:    function _assertCallerIsDriver(uint32 driverId) internal view {
    134:    function registerDriver(address driverAddr) public whenNotPaused returns (uint32 driverId) {
    145:    function driverAddress(uint32 driverId) public view returns (address driverAddr) {
    152:    function updateDriverAddress(uint32 driverId, address newDriverAddr) public whenNotPaused {
    160:    function nextDriverId() public view returns (uint32 driverId) {
    169:    function cycleSecs() public view returns (uint32 cycleSecs_) {
    199:        returns (uint32 cycles)
    216:    function receiveDripsResult(uint256 userId, IERC20 erc20, uint32 maxCycles)
    219:        returns (uint128 receivableAmt)
    238:    function receiveDrips(uint256 userId, IERC20 erc20, uint32 maxCycles)
    241:        returns (uint128 receivedAmt)
    276:    ) public whenNotPaused returns (uint128 amt) {
    303:    ) public view returns (uint128 amt) {
    317:    function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
    328:    function splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
    331:        returns (uint128 collectableAmt, uint128 splitAmt)
    358:        returns (uint128 collectableAmt, uint128 splitAmt)
    372:    function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
    390:        returns (uint128 amt)
    409:    function give(uint256 userId, uint256 receiver, IERC20 erc20, uint128 amt)
    438:            uint32 updateTime,
    439:            uint128 balance,
    440:            uint32 maxEnd
    464:        uint32 timestamp
    465:    ) public view returns (uint128 balance) {
    514:        int128 balanceDelta,
    517:        uint32 maxEndHint1,
    518:        uint32 maxEndHint2
    519:    ) public whenNotPaused onlyDriver(userId) returns (int128 realBalanceDelta) {
    521:            _increaseTotalBalance(erc20, uint128(balanceDelta));
    533:            erc20.safeTransferFrom(msg.sender, address(this), uint128(realBalanceDelta));
    535:            _decreaseTotalBalance(erc20, uint128(-realBalanceDelta));
    536:            erc20.safeTransfer(msg.sender, uint128(-realBalanceDelta));
    560:        uint32 updateTime,
    561:        uint32 maxEnd
    629:    function _increaseTotalBalance(IERC20 erc20, uint128 amt) internal {
    635:    function _decreaseTotalBalance(IERC20 erc20, uint128 amt) internal {
    643:        return uint160(address(erc20));

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L58
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L64
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L77
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L84
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L97
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L99
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L109
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L119
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L124
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L134
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L145
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L152
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L160
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L169
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L199
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L216
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L219
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L238
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L241
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L276
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L303
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L317
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L328
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L331
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L358
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L372
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L390
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L409
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L438
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L439
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L440
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L464
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L465
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L514
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L517
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L518
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L519
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L521
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L533
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L535
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L536
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L560
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L561
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L629
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L635
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L643

File: main/src/Splits.sol

    11:    uint32 weight;
    22:    uint32 internal constant _TOTAL_SPLITS_WEIGHT = 1_000_000;
    25:    uint256 internal constant _MAX_TOTAL_SPLITS_BALANCE = type(uint128).max;
    33:    event Collected(uint256 indexed userId, uint256 indexed assetId, uint128 collected);
    42:        uint256 indexed userId, uint256 indexed receiver, uint256 indexed assetId, uint128 amt
    49:    event Collectable(uint256 indexed userId, uint256 indexed assetId, uint128 amt);
    57:        uint256 indexed userId, uint256 indexed receiver, uint256 indexed assetId, uint128 amt
    71:    event SplitsReceiverSeen(bytes32 indexed receiversHash, uint256 indexed userId, uint32 weight);
    88:        uint128 splittable;
    90:        uint128 collectable;
    98:    function _addSplittable(uint256 userId, uint256 assetId, uint128 amt) internal {
    106:    function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
    117:    function _splitResult(uint256 userId, SplitsReceiver[] memory currReceivers, uint128 amount)
    120:        returns (uint128 collectableAmt, uint128 splitAmt)
    126:        uint32 splitsWeight = 0;
    130:        splitAmt = uint128((uint160(amount) * splitsWeight) / _TOTAL_SPLITS_WEIGHT);
    145:        returns (uint128 collectableAmt, uint128 splitAmt)
    157:        uint32 splitsWeight = 0;
    160:            uint128 currSplitAmt =
    161:                uint128((uint160(collectableAmt) * splitsWeight) / _TOTAL_SPLITS_WEIGHT - splitAmt);
    176:    function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
    185:    function _collect(uint256 userId, uint256 assetId) internal returns (uint128 amt) {
    199:    function _give(uint256 userId, uint256 receiver, uint256 assetId, uint128 amt) internal {
    228:        uint64 totalWeight = 0;
    233:            uint32 weight = receiver.weight;

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L11
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L22
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L25
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L33
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L42
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L49
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L57
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L71
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L88
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L90
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L98
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L106
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L117
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L120
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L126
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L130
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L145
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L157
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L160
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L161
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L176
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L185
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L199
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L228
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L233

File: main/src/NFTDriver.sol

    25:    uint32 public immutable driverId;
    32:    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
    113:        returns (uint128 amt)
    133:    function give(uint256 tokenId, uint256 receiver, IERC20 erc20, uint128 amt)
    193:        uint32 maxEndHint1,
    194:        uint32 maxEndHint2,
    198:            _transferFromCaller(erc20, uint128(balanceDelta));
    204:            erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
    285:    function _transferFromCaller(IERC20 erc20, uint128 amt) internal {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L25
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L32
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L113
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L133
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L193
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L194
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L198
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L204
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L285

File: main/src/AddressDriver.sol

    25:    uint32 public immutable driverId;
    30:    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
    41:        return (uint256(driverId) << 224) | uint160(userAddr);
    60:    function collect(IERC20 erc20, address transferTo) public whenNotPaused returns (uint128 amt) {
    76:    function give(uint256 receiver, IERC20 erc20, uint128 amt) public whenNotPaused {
    128:        uint32 maxEndHint1,
    129:        uint32 maxEndHint2,
    133:            _transferFromCaller(erc20, uint128(balanceDelta));
    145:            erc20.safeTransfer(transferTo, uint128(-realBalanceDelta));
    170:    function _transferFromCaller(IERC20 erc20, uint128 amt) internal {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L25
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L30
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L41
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L60
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L76
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L128
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L129
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L133
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L145
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L170

File: main/src/ImmutableSplitsDriver.sol

    15:    uint32 public immutable driverId;
    17:    uint32 public immutable totalSplitsWeight;
    28:     constructor(DripsHub _dripsHub, uint32 _driverId) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L15
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L17
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L28

## [G-02] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
It is not necessary to have both a named return and a return statement.
Contexts: 59
    File: main/src/Drips.sol

    63:        returns (DripsConfig)
    273:        returns (
    303:        returns (uint32 cycles)
    317:        returns (uint32 fromCycle, uint32 toCycle)
    401:        returns (
    448:    ) private pure returns (bytes32[] memory historyHashes) {
    475:    ) private view returns (uint128 squeezedAmt) {
    511:        returns (
    538:    ) internal view returns (uint128 balance) {
    561:    ) private view returns (uint128 balance) {
    685:    ) private view returns (uint32 maxEnd) {
    719:                    return uint32(end);
    742:    ) private view returns (bool isEnough) {
    772:        returns (uint256[] memory configs, uint256 configsLen)
    795:        returns (uint256 newConfigsLen)
    818:        returns (uint256 amtPerSec, uint256 start, uint256 end)
    837:        returns (bytes32 dripsHash)
    857:    ) internal pure returns (bytes32 dripsHistoryHash) {
    973:        returns (uint32 start, uint32 end)
    991:    ) private pure returns (uint32 start, uint32 end_) {
    1096:        returns (uint256 amt)
    1120:    function _cycleOf(uint32 timestamp) private view returns (uint32 cycle) {
    1128:    function _currTimestamp() private view returns (uint32 timestamp) {
    1134:    function _currCycleStart() private view returns (uint32 timestamp) {
    1142:    function _dripsStorage() private view returns (DripsStorage storage dripsStorage) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L60-L64
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L270-L279
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L300-L303
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L314-L317
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L392-L407
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L444-L448
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L470-L475
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L508-L517
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L533-L538
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L555-L561
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L680-L685
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L737-L742
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L769-L772
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L792-L795
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L815-L818
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L834-L837
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L851-L857
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L970-L973
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L985-L991
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1093-L1096
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1120
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1128
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1134
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1142

File: main/src/DripsHub.sol

    145:    function driverAddress(uint32 driverId) public view returns (address driverAddr) {
    160:    function nextDriverId() public view returns (uint32 driverId) {
    169:    function cycleSecs() public view returns (uint32 cycleSecs_) {
    181:    function totalBalance(IERC20 erc20) public view returns (uint256 balance) {
    199:        returns (uint32 cycles)
    219:        returns (uint128 receivableAmt)
    303:    ) public view returns (uint128 amt) {
    317:    function splittable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
    331:        returns (uint128 collectableAmt, uint128 splitAmt)
    372:    function collectable(uint256 userId, IERC20 erc20) public view returns (uint128 amt) {
    435:        returns (
    465:    ) public view returns (uint128 balance) {
    546:    function hashDrips(DripsReceiver[] memory receivers) public pure returns (bytes32 dripsHash) {
    562:    ) public pure returns (bytes32 dripsHistoryHash) {
    587:    function splitsHash(uint256 userId) public view returns (bytes32 currSplitsHash) {
    598:        returns (bytes32 receiversHash)
    621:    function _dripsHubStorage() internal view returns (DripsHubStorage storage storageRef) {
    642:    function _assetId(IERC20 erc20) internal pure returns (uint256 assetId) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L145
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L160
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L169
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L181
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L196-L199
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L216-L219
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L297-L303
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L317
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L328-L331
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L372
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L432-L441
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L460-L465
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L546
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L557-L562
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L587
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L595-L598
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L621
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L642

File: main/src/Splits.sol

    106:    function _splittable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
    120:        returns (uint128 collectableAmt, uint128 splitAmt)
    176:    function _collectable(uint256 userId, uint256 assetId) internal view returns (uint128 amt) {
    262:    function _splitsHash(uint256 userId) internal view returns (bytes32 currSplitsHash) {
    273:        returns (bytes32 receiversHash)
    283:    function _splitsStorage() private view returns (SplitsStorage storage splitsStorage) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L106
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L117-L120
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L176
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L262
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L270-L273
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L283

File: main/src/NFTDriver.sol

    50:    function nextTokenId() public view returns (uint256 tokenId) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L50

File: main/src/Managed.sol

    105:    function isPauser(address pauser) public view returns (bool isAddrPauser) {
    112:    function allPausers() public view returns (address[] memory pausersList) {
    117:    function paused() public view returns (bool isPaused) {
    136:    function _erc1967Slot(string memory name) internal pure returns (bytes32 slot) {
    142:    function _managedStorage() internal view returns (ManagedStorage storage storageRef) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L105
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L112
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L117
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L136
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L142

File: blob/main/src/Caller.sol

    124:    function isAuthorized(address sender, address user) public view returns (bool authorized) {
    132:    function allAuthorized(address sender) public view returns (address[] memory authorized) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L124
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L132

File: main/src/AddressDriver.sol

    40:    function calcUserId(address userAddr) public view returns (uint256 userId) {
    46:    function callerUserId() internal view returns (uint256 userId) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L40
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L46

File: main/src/ImmutableSplitsDriver.sol

    36:     function nextUserId() public view returns (uint256 userId) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L36

## [G-03] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.
Contexts: 01
    File: main/src/Drips.sol

    176:        mapping(uint256 => mapping(uint256 => DripsState)) states;
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L176

## [G-04] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance
Contexts: 07
File: main/src/Drips.sol

    219:    constructor(uint32 cycleSecs, bytes32 dripsStorageSlot) {
    220:        require(cycleSecs > 1, "Cycle length too low");
    221:        _cycleSecs = cycleSecs;
    222:        _dripsStorageSlot = dripsStorageSlot;
    223:    }

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L219-L223

File: main/src/DripsHub.sol

    109:     constructor(uint32 cycleSecs_)
    110        Drips(cycleSecs_, _erc1967Slot("eip1967.drips.storage"))
    111        Splits(_erc1967Slot("eip1967.splits.storage"))
    112    {
    113        return;
    114    }

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L109-L114

File: main/src/Splits.sol

    94:    constructor(bytes32 splitsStorageSlot) {
    95:        _splitsStorageSlot = splitsStorageSlot;
    96:    }

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L94-L96

File: main/src/NFTDriver.sol

    32:    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
    33:        ERC2771Context(forwarder)
    34:        ERC721("DripsHub identity", "DHI")
    35:    {
    36:        dripsHub = _dripsHub;
    37:        driverId = _driverId;
    38:    }

    158:    constructor(Managed logic, address admin) ERC1967Proxy(address(logic), new bytes(0)) {
    159:        _changeAdmin(admin);
    160:    }

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L32-L38
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Managed.sol#L158-L160

File: main/src/AddressDriver.sol

    30:    constructor(DripsHub _dripsHub, address forwarder, uint32 _driverId)
    31:        ERC2771Context(forwarder)
    32:    {
    33:        dripsHub = _dripsHub;
    34:        driverId = _driverId;
    35:    }

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L30-L35

File: main/src/ImmutableSplitsDriver.sol

    28:    constructor(DripsHub _dripsHub, uint32 _driverId) {
    29:        dripsHub = _dripsHub;
    30:        driverId = _driverId;
    31:        totalSplitsWeight = _dripsHub.TOTAL_SPLITS_WEIGHT();
    32:    }

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L28-L32

## [G-05] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS:
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.
Contexts: 14
    File: main/src/Drips.sol

    247:            for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
    287:        for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
    358:        for (uint256 i = 0; i < squeezedNum; i++) {
    422:        for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {
    450:        for (uint256 i = 0; i < dripsHistory.length; i++) {
    490:        for (; idx < receivers.length; idx++) {
    563:        for (uint256 i = 0; i < receivers.length; i++) {
    664:            for (uint256 i = 0; i < newReceivers.length; i++) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L247
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L287
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L358
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L422
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L450
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L490
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L563
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L664

File: main/src/DripsHub.sol

    613:        for (uint256 i = 0; i < userMetadata.length; i++) {
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L613

File: main/src/Splits.sol

    127:        for (uint256 i = 0; i < currReceivers.length; i++) {
    158:        for (uint256 i = 0; i < currReceivers.length; i++) {
    231:        for (uint256 i = 0; i < receivers.length; i++) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L127
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L158
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L231

File: main/src/Caller.sol

    196:        for (uint256 i = 0; i < calls.length; i++) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L196

File: main/src/ImmutableSplitsDriver.sol

    61:        for (uint256 i = 0; i < receivers.length; i++) {

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L61

## [G-06] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
Contexts: 21
    File: main/src/Drips.sol

    253:                amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
    284:            toCycle -= receivableCycles;
    289:            amtPerCycle += state.amtDeltas[cycle].thisCycle;
    290:            receivedAmt += uint128(amtPerCycle);
    291:            amtPerCycle += state.amtDeltas[cycle].nextCycle;
    429:                    amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);
    495:            amt += _drippedAmt(receiver.config.amtPerSec(), start, end);
    572:            balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
    755:                spent += _drippedAmt(amtPerSec, start, end);
    1053:            amtDelta.thisCycle += int128(fullCycle - nextCycle);
    1054:            amtDelta.nextCycle += int128(nextCycle);

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L253
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L284
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L288
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L289
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L290
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L429
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L495
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L572
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L755
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1053
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1054

File: main/src/DripsHub.sol

    632:        totalBalances[erc20] += amt;
    636:        _dripsHubStorage().totalBalances[erc20] -= amt;

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L632
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L636

File: main/src/Splits.sol

    99:        _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
    128:            splitsWeight += currReceivers[i].weight;
    159:            splitsWeight += currReceivers[i].weight;
    162:            splitAmt += currSplitAmt;
    167:        collectableAmt -= splitAmt;
    168:        balance.collectable += collectableAmt;
    235:            totalWeight += weight;

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L99
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L128
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L159
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L162
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L167
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L168
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L235

File: main/src/ImmutableSplitsDriver.sol

    62:            weightSum += receivers[i].weight;

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/ImmutableSplitsDriver.sol#L62

## [G-07] ADDED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT
EX: require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
Contexts: 06
    File: main/src/Drips.sol

    283:            receivableCycles = toCycle - fromCycle - maxCycles;
    284:            toCycle -= receivableCycles;
    423:            DripsHistory memory drips = dripsHistory[dripsHistory.length - i];
    425:                uint32 squeezeStartCap = nextSqueezed[currCycleConfigs - i];

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L283
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L284
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L423
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L425

File: main/src/Splits.sol

    131:        collectableAmt = amount - splitAmt;
    167:        collectableAmt -= splitAmt;

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L131
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L167

## [G-08] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.
Contexts: 09
    File: main/src/Drips.sol

    355:        bytes32[] memory squeezedHistoryHashes = new bytes32[](squeezedNum);
    423:            DripsHistory memory drips = dripsHistory[dripsHistory.length - i];
    451:            DripsHistory memory drips = dripsHistory[i];
    476:        DripsReceiver[] memory receivers = dripsHistory.receivers;
    491:            DripsReceiver memory receiver = receivers[idx];
    564:            DripsReceiver memory receiver = receivers[i];
    665:                DripsReceiver memory receiver = newReceivers[i];
    778:                DripsReceiver memory receiver = receivers[i];

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L355
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L423
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L451
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L476
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L491
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L564
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L665
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L778

File: main/src/Caller.sol

    197:            Call memory call = calls[i];

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L197

## [G-09] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT
Contexts: 01
File: blob/main/src/Caller.sol

    82:    string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("
    83        "address sender,address to,bytes data,uint256 value,uint256 nonce,uint256 deadline)";
    84:    bytes32 internal immutable callSignedTypeHash = keccak256(bytes(CALL_SIGNED_TYPE_NAME));

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L82-L84    

## [G-10] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are
CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost
Contexts: 10
File: main/src/DripsHub.sol

    389:         onlyDriver(userId)
    412:        onlyDriver(userId)
    519:    ) public whenNotPaused onlyDriver(userId) returns (int128 realBalanceDelta) {
    579:        onlyDriver(userId)
    611:        onlyDriver(userId)

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L386-L390
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L409-L412
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L510-L519
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L576-L582
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L608-L617

File: main/src/NFTDriver.sol

    112:        onlyHolder(tokenId)
    136:        onlyHolder(tokenId)
    196:    ) public whenNotPaused onlyHolder(tokenId) returns (int128 realBalanceDelta) {
    223:        onlyHolder(tokenId)
    238:        onlyHolder(tokenId)

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L109-L113
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L133-L140
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L186-L196
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L220-L226
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L235-L241

## [G‑11] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.
Contexts: 02
File: main/src/DripsHub.sol

    221:        (receivableAmt,,,,) = Drips._receiveDripsResult(userId, _assetId(erc20), maxCycles);
    304:        (amt,,,,) =
    305:            Drips._squeezeDripsResult(userId, _assetId(erc20), senderId, historyHash, dripsHistory);

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L221
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L304-L305

## [G-12] USE BYTES32 INSTEAD OF STRING
Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.
Contexts: 01
File: blob/main/src/Caller.sol

    82:    string internal constant CALL_SIGNED_TYPE_NAME = "CallSigned("

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L82

