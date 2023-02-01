# Non-critical

* Use of negation can lead to unexpected reverts
* Comment makes plural reference but argument is singular
* Inconsistent return value naming
* Many functions with return values lack natspec comments for them
* Consider renaming `Managed.paused()` to `isPaused()`


### Use of negation can lead to unexpected reverts

The codebase uses the unary negation operator (`-`) in several locations dealing with int128 values. These representations are very large at the `MIN_INT` value and are unlikely to ever reach that quantity in production.

However, if somehow the value was ever reached, the process would cause a revert, as `MIN_INT` cannot be inverted. Tokens that use more than 18 decimals for representation mildly increase the risks.

Optimally, a function would be used in place of the negation that tests for the `MIN_INT` value and only inverts the sign if it would not revert, and perform some other non-reverting action if it would.

---

### Comment makes plural reference but argument is singular

```
/// @notice Preprocess and add a drips receiver to the list of configurations.
/// @param configs The list of drips configurations
/// @param configsLen The length of `configs`
/// @param receiver The added drips receivers. 
/// @return newConfigsLen The new length of `configs`
function _addConfig(uint256[] memory configs, uint256 configsLen, DripsReceiver memory receiver) private view returns (uint256 newConfigsLen) {
    uint256 amtPerSec = receiver.config.amtPerSec();
    require(amtPerSec != 0, "Drips receiver amtPerSec is zero");
    (uint256 start, uint256 end) = _dripsRangeInFuture(receiver, _currTimestamp(), type(uint32).max);
    // slither-disable-next-line incorrect-equality,timestamp
    if (start == end) {
        return configsLen;
    }
    configs[configsLen] = (amtPerSec << 64) | (start << 32) | end;
    return configsLen + 1;
}
```

The `receiver` param comment uses the plural sense but the function argument is a singular value, not an array.

---

### Inconsistent return value naming

```
function _dripsRange(DripsReceiver memory receiver, uint32 updateTime, uint32 maxEnd, uint32 startCap, uint32 endCap)
    private
    pure
    returns (uint32 start, uint32 end_)
```

The value `end_` is used instead of just `end` because `end` is used as a temporary value inside the function. For consistency, consider swapping the names.

---

### Many functions with return values lack natspec comments for them

The following functions have return values without a corresponding natspec comment (@return) for them:

`Managed.paused`
all functions in `DripsConfigImpl`
`Drips._dripsRangeInFuture`
`Drips._dripsRange`
`Drips._isOrdered`
`Drips._drippedAmt`
`Caller._call` lacks any comments
`NFTDriver._msgSender` lacks any comments
`NFTDriver._msgData` lacks any comments

---

### Consider renaming `Managed.paused()` to `isPaused()`

Using `isPaused` is a more conventional semantic meaning over simply `paused`.