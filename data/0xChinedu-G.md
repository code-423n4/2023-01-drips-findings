# [G-01] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance

*There are 7 instances of this:*

File: src/AddessDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol/#L30

File: src/Drips/sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L219

File: src/DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol/#L109

File: src/ImmutableSplitsDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol/#L28

File: src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol/#L73

File: src/NFTDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol/#L32

File: src/Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol/#L94


# [G-02] ++i/i++ SHOULD BE UNCHECKED{++i}/UNCHECKED{i++} WHEN IT IS NOT POSSIBLE
FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR AND WHILE LOOPS
The unchecked keyword is new in Solidity version 0.8.0, so this applies to that version
or higher, which these instances are. This saves 30-40 gas per loop.

*There are 14 instances of this:*

File: src/Caller.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol/#L196

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L247

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L287 

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L358

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L422

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L450

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L490

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L563

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L664

File: src/DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol/#L613

File: src/ImmutableSplitsDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol/#L61

File: src/Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol/#L127

File: src/Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol/#L158

File: src/Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol/#L231


# [G-03] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE
MARKED PAYABLE
If a function modifier such as 'onlyOwner' is used (in this case 'onlyHolder', 'onlyAdmin', 'onlyDriver'), the function will revert
if a normal user tries to pay the function. Marking the function as payable
will lower the cost of gas for legitimate callers because the compiler will
not include checks for whether a payment was provided.

*There are 13 instances of this:*

File: src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol/L#84

File: src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol/L#90

File: src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol/L#97

File: src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol/L#122

File: src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol/L#128

File: src/Managed.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol/L#151

File: src/NFTDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol/#L112

File: src/NFTDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol/#L136

File: src/NFTDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol/#L223

File: src/NFTDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol/#L196

File: src/NFTDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol/#L238

File: src/DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol/#L389

File: src/DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol/#L611


# [G-04] USING UNCHECKED BLOCKS TO SAVE GAS
Solidity version 0.8+, comes with implicit overflow and underflow checks on
unsigned integers. When an overflow or an underflow isn't possible (as an example, when a comparison
is made before the arithmetic operation), some gas can be saved by using an unchecked block.

*There are 6 instances of this:*

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L283

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L482

File: src/DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol/#L632

File: src/ImmutableSplitsDriver.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol/#L62

File: src/Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol/#L131


File: src/Splits.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol/#L161


# [G-05] x += y COSTS MORE GAS THAN x = x + y FOR STATE VARIABLES

*There are 2 instances of this:*

File: src/Drips.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol/#L253

File: src/DripsHub.sol
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol/#L632

