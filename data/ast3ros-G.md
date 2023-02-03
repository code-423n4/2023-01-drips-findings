# [GAS-1] Use uint256 instead of uint128 to save gas in calculation

As the EVM operates on 32 bytes at a time, variables smaller than that get converted. If we are not saving gas by packing the variable, it is cheaper for us to use 32 byte data types such as uint256.

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

Instance:

The total balance is uint256. The amount should be used as uint256 instead of uint128
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L629
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L635
The return of collect function should use uint256 as well
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L390


# [GAS-2] Check if the amount = 0 before state update and external call to save gas
If the amount calculated by Splits._collect = 0, revert early to save gas from additional state update and external call

Instance:
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L392

# [GAS-3] Emit event ReceivedDrips even though there is no state change.
When the receivedAmt is calculated to be zero or fromCycle = toCylce, there is no state change. However the event ReceivedDrips is still emitted.
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L256

When `newSplitsHash == state.splitsHash`, there is no state channge. However the event SplitsSet is still emitted
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L212

The event should be put in the if code block to save gas.
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L243-L254

# [GAS-4] Caching state variables in local/memory variables avoids SLOADs to save gas
The state.amtDeltas[cycle] variable should be cached in memory to avoid SLOAD multiple times.
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L286
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L288
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L290

# [GAS-5] Use uint32 in timestamp to avoid conversion and save gas
The `end` variable is uint40, even though `start` and `duration` variables are uint32. It requires two additional conversions:
- convert start annd duration to uint40
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L997
- convert end from uint40 to uint32
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Drips.sol#L1012

# [GAS-6] Avoid override transferFrom/safeTransferFrom/burn when adding modifier whenNotPause
The functions transferFrom/safeTranferFrom/burn are overrided to add whenNotPause modifier. However it leads to higher deployment cost with additional functions.
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L254
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L263
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L277
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/NFTDriver.sol#L244

Add modifier in _beforeTokenTransfer is sufficient.

    function _beforeTokenTransfer(address from, address to, uint256 tokenId, uint256 batchSize)
        internal
        whenNotPaused
        override
    {
        super._beforeTokenTransfer(from, to, tokenId, batchSize);
    }

# [GAS-7] Merge two require statements into one to save gas:
There are two require statements to check if splits receivers are duplicated or not sorted. It can be merged into one with the message: invalid split receivers
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/Splits.sol#L238-L239