Drips.sol line: 287 in function _receiveDripsResult(uint256 userId, uint256 assetId, uint32 maxCycles)
```
        for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
            amtPerCycle += state.amtDeltas[cycle].thisCycle;
            receivedAmt += uint128(amtPerCycle);
            amtPerCycle += state.amtDeltas[cycle].nextCycle;
        }
```
consider doing the addition in an ```unchecked``` block.
```
        for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
            unchecked{
                amtPerCycle += state.amtDeltas[cycle].thisCycle;
                receivedAmt += uint128(amtPerCycle);
                amtPerCycle += state.amtDeltas[cycle].nextCycle;
                }
        }
```