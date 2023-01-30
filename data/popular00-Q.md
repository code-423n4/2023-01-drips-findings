# General Comments
- Clean and elegant codebase, extensive testing, clearly commented. Nice!

# QA / Informational

- SafeERC20.safeApprove() is deprecated, but no issue w/ usage here
- Typo in `Caller.sol - callSigned()` comments - "Requires a `sender`'s signature of an ERC-721 message approving the call."; Likely meant EIP-712 rather than ERC-721

### `NFTDriver.sol` 
- Probably intended design but worth mentioning:
   + If a user sets up a drip and then sells their token, the new holder can drain whatever drips the original holder had set up. If this is intended design, I would recommend making this clear on the frontend.
   + If a user burns their NFT, all funds currently collectable + any future funds for that token will be locked, and there is nothing preventing a user from erroneously dripping to a burnt NFT (i.e. a `userId` equivalent of an `address(0)` check). 
