# 1. Uncheck arithmetics operations that can’t underflow/overflow

When an overflow or an underflow isn’t possible, some gas can be saved using an unchecked block.

For example on the [lines 563-573](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L563-L573) you would refactor it like so:

```diff
File: /src/Drips.sol
-   for (uint256 i = 0; i < receivers.length; i++) {
+   for (uint256 i = 0; i < receivers.length;) {
        DripsReceiver memory receiver = receivers[i];
        (uint32 start, uint32 end) = _dripsRange({
            receiver: receiver,
            updateTime: lastUpdate,
            maxEnd: maxEnd,
            startCap: lastUpdate,
            endCap: timestamp
        });
        balance -= uint128(_drippedAmt(receiver.config.amtPerSec(), start, end));

+       unchecked { i++ }
    }
```

Affected line of code:

- [Drips.sol#L450](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L450)

- [Drips.sol#L563](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L563)
- [Drips.sol#L664](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L664)
- [Drips.sol#L777](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L777)
- [Splits.sol#L127](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L127)

# 2. Using `storage` instead of `memory` for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

As an example on [line 197](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L197) you would refactor it like so:

```solidity
File: /src/Caller.sol
Line 197:    Call storage call = calls[i];
```

Affected lines of code:

- [Caller.sol#L197](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Caller.sol#L197)

- [Drips.sol#L423](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L423)

- [Drips.sol#L451](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L451)

- [Drips.sol#L476](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L476)

- [Drips.sol#L491](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L491)

- [Drips.sol#L564](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L564)

- [Drips.sol#L665](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L665)

- [Drips.sol#L778](https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L778)

- [Splits.sol#L232](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L232)

# 3. Regular addition/subtraction assignment saves gas

Using standard addition or subtraction assignment (`x = x + y` or `x = x - y`) rather than the shorthand versions (`x += y` or `x -= y`) saves gas. This can save approximately 22 gas per occurrence.

For example on [lines 490-496](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L490-L496) may be refactored as:

```diff
File: /src/Drips.sol
    for (; idx < receivers.length; idx++) {
        DripsReceiver memory receiver = receivers[idx];
        if (receiver.userId != userId) break;
        (uint32 start, uint32 end) =
            _dripsRange(receiver, updateTime, maxEnd, squeezeStartCap, squeezeEndCap);
-       amt += _drippedAmt(receiver.config.amtPerSec(), start, end);
+       amt = amt + _drippedAmt(receiver.config.amtPerSec(), start, end);
    }
```

Affected line of code:

- [Drips.sol#L495](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L495)
- [Drips.sol#L572](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L572)
- [Drips.sol#L755](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L755)
- [Splits.sol#L128](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L128)

# 4. `>`/`<` Is Cheaper Than `>=`/`<=`

Using the `>`/`<` operator requires fewer opcodes and therefore costs less gas than using the `>=`/`<=` operator. This is demonstrated in the tweet linked below:

https://twitter.com/GalloDaSballo/status/1543729467465629696

This can be applied to the following line of code:

- [Drips.sol#L748](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L748)
- [Drips.sol#L540](https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L540)
