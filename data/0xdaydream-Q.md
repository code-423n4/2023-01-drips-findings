Using compound assignment operators for state variables (like State += X or State -= X) is more expensive than using operator assignment (like State = State + Xor State = State - X).


+ Splits.sol:99:        _splitsStorage().splitsStates[userId].balances[assetId].splittable += amt;
+ Splits.sol:128:            splitsWeight += currReceivers[i].weight;
+ Splits.sol:159:            splitsWeight += currReceivers[i].weight;
+ Splits.sol:162:            splitAmt += currSplitAmt;
+ Splits.sol:167:        collectableAmt -= splitAmt;
+ Splits.sol:168:        balance.collectable += collectableAmt;
+ Splits.sol:235:            totalWeight += weight;
+ DripsHub.sol:632:        totalBalances[erc20] += amt;
+ DripsHub.sol:636:        _dripsHubStorage().totalBalances[erc20] -= amt;
+ Drips.sol:253:                amtDeltas[toCycle].thisCycle += finalAmtPerCycle;
+ Drips.sol:284:            toCycle -= receivableCycles;
+ Drips.sol:288:            amtPerCycle += state.amtDeltas[cycle].thisCycle;
+ Drips.sol:289:            receivedAmt += uint128(amtPerCycle);
+ Drips.sol:290:            amtPerCycle += state.amtDeltas[cycle].nextCycle;
+ Drips.sol:429:                    amt += _squeezedAmt(userId, drips, squeezeStartCap, squeezeEndCap);
+ Drips.sol:495:            amt += _drippedAmt(receiver.config.amtPerSec(), start, end);
+ Drips.sol:572:            balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));
+ Drips.sol:755:                spent += _drippedAmt(amtPerSec, start, end);
+ Drips.sol:1053:            amtDelta.thisCycle += int128(fullCycle - nextCycle);
+ Drips.sol:1054:            amtDelta.nextCycle += int128(nextCycle);
+ ImmutableSplitsDriver.sol:62:            weightSum += receivers[i].weight;