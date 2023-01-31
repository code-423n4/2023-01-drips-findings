**Drips.sol**
- L454/540 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS

- L283/284/426/634 - A subtraction is performed between two uints, but in line 282/426/633 the check is already performed indirectly, therefore unchecked can be used and not perform unnecessary extra validation.

- L356/357/419/425/487/488/494/923/935/938/946/1045 - A variable is created in memory but it is only used once, therefore creating it generates an unnecessary extra gas cost, it should be eliminated and used directly where the operation is needed.

- L361/366/482/780/806/1122 - Instead of doing i - 1 or i + 1 you can do ++i or --i generating a lower cost of gas.


**Splits.sol**
- L148 - A variable is created in memory but it is only used once, therefore creating it generates an unnecessary extra gas cost, it should be eliminated and used directly where the operation is needed.

- L239 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS


**NFTDriver.sol**
- L43 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS


**Managed.sol**
- L54 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS

- L174/175/180 - A variable is created in memory but it is only used once, therefore creating it generates an unnecessary extra gas cost, it should be eliminated and used directly where the operation is needed.
