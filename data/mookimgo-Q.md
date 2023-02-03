# 1. incompatible with fee-on-transfer tokens and rebasing tokens

For those tokens with fee enabled for transfering or balance-rebasing tokens, this protocol will not work.
This should be noted on docs, instead of claiming that `Drips v2 allows streaming of any ERC20 token.`

docs link: https://v2.docs.drips.network/docs/drips-v2-features

The comment in the code verify this:
https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L55

    /// Tokens which rebase the holders' balances, collect taxes on transfers,
    /// or impose any restrictions on holding or transferring tokens are not supported.
    /// If you use such tokens in the protocol, they can get stuck or lost.

Suggestion: update docs to clarify this to avoid misunderstanding to users