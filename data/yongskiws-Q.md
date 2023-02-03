## _addDelta  & _addDeltaRange lacks validation to prevent any over- or under-flows

## Impact
data type int256 which has a lower bound of -2^255 and an upper bound of 2^255-1. If the result of an arithmetic operation exceeds upper limit, underflow will occur lacks validation the ` _addDelta` function to call from the `_addDeltaRange` will cause underflow or the result of an arithmetic operation that exceeds the lower or upper limit of the data type int256 can cause unexpected results and logical errors and `_addDeltaRange` still has the potential to over- or under-flows

## Proof of Concept
The addition of lacks validation ensures that the arithmetic operations performed do not produce underflow or results that exceed the int128 data type limit. However, the `_addDeltaRange` function still has the potential to overflow or underflow.it is necessary to add checks to ensure that the amtPerSec input does not exceed the int256 data type limit or result in underflow.

## Tools Used
Manual Review 

## Recommended Mitigation Steps
need to add a lacks validation ensuring that the amtPerSec input does not exceed the int256 data type limit or result in underflow in the `_addDeltaRange function`
as an example
``` solidity
require(amtPerSec >= INT256_MIN && amtPerSec <= INT256_MAX, "Input amtPerSec over- or under-flows") ///  before calling the _addDelta function.
``` 
