1. using unchecked for increase counting in loops saves gas
* Drips.sol#872 :: _updateReceiverStates :: L958 

>*Gas Report Before:*
>    forge test --match-contract "DripsHub" --gas-report | grep setDrips
>    | setDrips                           | 1790            | 120849 | 122841 | 287693 | 16      |
>    | setDrips                              | 2231            | 121564 | 123278 | 288148 | 16      |
>*After:* 
>    forge test --match-contract "DripsHub" --gas-report | grep setDrips
>    | setDrips                           | 1790            | 120815 | 122811 | 287573 | 16      |
>    | setDrips                              | 2231            | 121531 | 123248 | 288028 | 16      |

```
//OLD 
if (pickCurr) {
   currIdx++;
}
if (pickNew) {
	newIdx++;
}
//NEW
if (pickCurr) {
	unchecked {++currIdx;}
}
if (pickNew) {
  unchecked {++newIdx;}
}
```


2.0. **Using calldata instead of memory in external functions saves gas**

2.1. DripsHub.sol#510 :: setDrips 
Gas Report Before
>	❯ forge test --match-contract "DripsHub" --gas-report | grep setDrips
>	| setDrips                           | 1790            | 120849 | 122841 | 287693 | 16      |
>	| setDrips                              | 2231            | 121564 | 123278 | 288148 | 16      |
After:
>	❯ forge test --match-contract "DripsHub" --gas-report | grep setDrips
>	| setDrips                           | 1389            | 120738 | 122764 | 287856 | 16      |
>	| setDrips                              | 1830            | 121454 | 123201 | 288311 | 16      |




2.3. DripsHub.sol#576 :: setSplits
Gas Report Before 
>	❯ forge test --match-contract "DripsHub" --gas-report | grep setSplits
>	| setSplits                          | 964             | 15756  | 16103  | 30053  | 6       |
>	| setSplits                             | 1369            | 16910  | 18312  | 30457  | 6       |
After:
>	❯ forge test --match-contract "DripsHub" --gas-report | grep setSplits
>	| setSplits                          | 759             | 15744  | 16188  | 30138  | 6       |
>	| setSplits                             | 1164            | 16899  | 18252  | 30542  | 6       |


2.3. DripsHub.sol#355 :: split 
Gas Report Before
>	❯ forge test --match-contract "DripsHub" --gas-report | grep "split\s"
>	| split                              | 1107            | 14320  | 19611  | 40776  | 21      |
>	| split                                 | 1518            | 14672  | 19934  | 41108  | 21      |
After:
>	❯ forge test --match-contract "DripsHub" --gas-report | grep "split\s"
>	| split                              | 899             | 14252  | 19541  | 40796  | 21      |
>	| split                                 | 1310            | 14604  | 19864  | 41128  | 21      |

