unused import will cost more gas
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L5
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L4 DripsConfigImpl
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L4 DripsHistory

use uint 256 instead. When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher.
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L25 30 128 129 

Use calldata instead of memory for function arguments that do not get mutated
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L202

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops 
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L196
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L614
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L247 287 358 479 490 422 450 563 664 745 777
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L61
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L232 127 158 

Using storage instead of memory for structs/arrays saves gas
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193 132
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L347 350, 351,355,397,404,405,446,448,476,536,559,613,615,682,687,739,769,772,792,815,834,874,877,
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L275 302 328 355  463 513 515 546 576 595
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L112
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L117 143 212 226 250 270

should be imumtable
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L112

use uint 256 instead of bool to save gas
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L883 94 742 883 890 1064

Can make the variable outside the loop to save gas. Make it outside and only use it inside.
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L361 425 452 480 493 565 665 746 
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L197
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L614
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L232 233 236 160 161 163 

avoid Using compound assignment operators to save gas
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632 636 625
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L253 288-290 429 495 755 1053 1054 284 572
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L62
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L167 99 128 159 162 168 235 

Not using the named return variables when a function returns, wastes deployment gas
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L36
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L145 160 169 181 199 317 331 358 372 436-440 465 546 562 587 598 642 
