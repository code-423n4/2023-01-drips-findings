## User might loose ETH in `callBatched()` if they send more than needed
`callBatched()` doesn't verify that the amount of value the user sent matches the amount of value. In case a user sent more ETH than spent in the calls it can be stolen by another user.
Consider the following scenario:
* Bob wants to drip WETH to some receivers, so he sends some ETH to convert it to WETH
* Bob intended to convert 0.1 ETH, but set the `msg.value` to 1 ETH
* Alice operates a bot that monitors the `Caller` address, and right away executes via `callBatched()` a call that would simply transfer the remaining 0.9 ETH to her
As a result, Bob lost 0.9 ETH do to a typo

### Mitigation
Verify that `msg.value` is equal to the total value used in the batched calls.
You can also add an unsafe version `callBatchedUnsafe()` for users who want to save that extra gas, or for users who are worried that an insignificant difference would cause a revert.

## FoT and rebase tokens
It should be noted that those kind of tokens can cause issues, since the contract might end up having less balance than it assumes it has, and the last users to collect their funds wouldn't be able to do it.

## Using `uint32` for timestamp might cause issues at 2106
This has been raised in past contest and isn't considered a significant issue (since projects usually don't care about 85 years from now), but it's worth mentioning it.
(this would cause issues mostly about redeeming old drips after that timestamp has past)




## At `ImmutableSplitsDriver` if user sends to himself the funds will be stuck forever
Not sure to what extent the protocol needs to protect users from dumb mistakes, but it might be worth adding a check at `ImmutableSplitsDriver.createSplits()` that the user isn't sending to himself, since in that case there will always be some funds that will be stuck in the user's `splittable` balance.

## `NFTDriver.burn()` might cause the user to loose access to funds
The `burn()` function here doesn't seem to be very useful, the only case it can be useful is to make a permanent splittable receivers, and for that we have already `ImmutableSplitsDriver`.
Consider removing it, or adding checks that the user splits config sends it all to other users.