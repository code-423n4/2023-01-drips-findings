## Non-critical issues
## [N-01] Use a more recent version of solidity

For security, it is best practice to use the latest Solidity version.

An old version is used (0.8.17), it is recommended to use the latest version (0.8.18)

foundry.toml :
```
[profile.default]
solc_version = '0.8.17'
optimizer_runs = 7_000
verbosity = 1
fuzz_runs = 5
[fmt]
line_length = 100
[fuzz]
runs = 4
```

---
## Low risk issues
## [L-01] Use Two-Step Transfer Pattern for Access Controls  

Contracts implementing access control's, e.g. owner, should consider implementing a Two-Step Transfer pattern.

Otherwise it's possible that the role mistakenly transfers ownership to the wrong address, resulting in a loss of the role.

*There is 1 instance of this issue*
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L84-L86
