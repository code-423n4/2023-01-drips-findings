# [Q-01] Conflicts between documentation and codebase function comments.

In the [documentation](https://v2.docs.drips.network/docs/drips-v2-features#multi-token-support-any-erc20) clearly says that: 
>Drips v2 allows streaming of any ERC20 token.

But in [DripsHub.sol](https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L403-L407) says:

>  It must preserve amounts, so if some amount of tokens is transferred to an address, then later the same amount must be transferrable from that address. Tokens which rebase the holders' balances, collect taxes on transfers, or impose any restrictions on holding or transferring **TOKENS ARE NOT SUPPORTED**. If you use such tokens in the protocol, they can get stuck or lost.