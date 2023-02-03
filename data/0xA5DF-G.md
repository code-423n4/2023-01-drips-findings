## Use a 'bitmap' to mark which `amtDeltas` entries have non-zero values
This one is significant and can save **millions of gas units**, esp. if the protocol uses a short cycle.
The only downside here is that it will increase the gas cost of setting drips.

The idea is taken from [Uniswap V3](https://github.com/Uniswap/v3-core/blob/main/contracts/libraries/TickBitmap.sol). The idea is to use a uint256 variable to mark which `amtDelta` entries are 'full' (i.e. have non-zero values).
This way, if we call `receiveDrips()` over 1000 cycles and only 8 of them have deltas - instead of reading 1000 slots we need to read only 12 slots (8 deltas + 4 bitmap entries).
In other words 25.2K gas instead of 2.1M (!).
(To be more precise the deletion of the entries gives us some gas refund, so we'll actually by paying ~2.4K instead of ~2.9M)

From sender's point the gas will increase by the following (per `amtDelta` written):
* If the `amtDelta` entries are already non-zero - than it'd cost only 100 gas units (there's no need to read the bitmap entry, just to read the `amtDelta` entry before writing to it)
* If the `amtDelta` entry is zero:
    * If the bitmap entry is non-zero it'll cost 5K
        * Unless that entry was already modified in the current tx, in that case it'll be only 100
    * If the bitmap entry is zero it'll cost 22.1K
    
Since each receiver adds 1 delta range that modifies 2 amtDelta entries that means 200 to 44.2K gas units per receiver. On the average case though it'd probably be 200-10K.
The more activity the receiving user will have the more it'll decrease the cost for the senders (senders will probably also converge around the cycles of other users in order to save gas for them).

On the other hand, for receivers with less activity it might be costly for the senders but will also save much more for receivers (since they have more `amtDelta` empty entries).

If a cycle of a week will be chosen it probably wouldn't be that relevant, but in case of shorter cycles (1 hour or 1 day) it can save lots of gas.

## Don't delete empty `amtDelta` entries
Gas saved: 100 units times the amount of empty entries

(this is in case the above suggestion isn't accepted)
At `_receiveDrips()` [all old entries are deleted](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L248), even the empty ones.
This imposes an additional 100 gas units cost per empty entry.
I think the simplest solution might be to implement a non-view version of `_receiveDripsResult()` that will delete non-empty entries.

## Read `amtDelta` to memory 
Gas saved: ~100 units per cycle

At `_receiveDripsResult()` each `amtDelta` entry is simply a single slot, reading the different fields of this struct separately takes an additional `sload` instruction that costs about 100 gas units (plus some additional cost to calculate the slot value).
Reading the whole struct to memory and reading from there can save that gas.

```diff
         for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {
-            amtPerCycle += state.amtDeltas[cycle].thisCycle;
+            AmtDelta memory amtDelta = state.amtDeltas[cycle];
+            amtPerCycle += amtDelta.thisCycle;
             receivedAmt += uint128(amtPerCycle);
-            amtPerCycle += state.amtDeltas[cycle].nextCycle;
+            amtPerCycle += amtDelta.nextCycle;
         }
```

Note: this should also be relevant to the `_addDelta()`, but for some reason the test results didn't show a decrease in gas cost. I didn't have the time to get the bottom of it and it might be worth looking into it.

## Cache `state.nextReceivableCycle`
`state.nextReceivableCycle` can be read twice [here](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L953).
It should be cached to memory first in order to save ~100 gas units.

## Have a built-in address driver
Using any driver costs an additional `sload` per tx - since it requires the `DripsHub` contract to check the `driverAddresses` map + the cost of calling an additional contract (calling a cold address - 2.1K units).
It may be worth to embed the functionality of common drivers like the address driver (reserve a driver ID to it, and at `_assertCallerIsDriver()` send it to the embedded functionality), or alternately register the driver's address in an immutable variable.
This might cost a few extra units (probably 10-20, just an additional condition check) for other drivers, but might save a significant amount of gas for common drivers.

