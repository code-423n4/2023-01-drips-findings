QA Report

ISSUE 1 - ++i/i++ Should Be unchecked{++i}/unchecked{i++} When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops 

Files 

	Caller.sol - 196
	Drips.sol - 247,287,358,422,450,490,563,664,745,777
	DripsHub.sol - 613
	ImmutableSplitsDriver.sol - 61
	Splits.sol - 127,158,231

------------------------------------------------------------------------------------------------

ISSUE 2 - require()/revert() Strings Longer Than 32 Bytes Cost Extra Gas

Files 

	Caller.sol - 108,116
	Drips.sol - 454,461,540,541,622,775,780,798
	DripsHub.sol - 125,631
	ImmutableSplitsDriver.sol - 64
	Managed.sol - 47,53,91,98	
	NFTDriver.sol - 41
	Splits.sol - 227,234,239,238,244

-----------------------------------------------------------------------------------------------

ISSUE 3 - Public Functions To External

Files

	AddressDriver.sol - L76,122,158