### **Low Risk Issues**
#### **[L-01] Driver Registration Frontrunning**
##### **Context**:
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30

##### **Description:**
`driverId` is passed as a constructor argument of new driver contracts in this repository. So unless the given id is already registered to a separate contract controlled by the deployer, or they are registering a counterfactual addreess, then DoS could occur by front-running driver registration such that `driverId` on the driver is different to that on DripsHub and calls from the driver are deemed invalid.

##### **Recommendation**:
Recommend drivers call `registerDriver` on `DripsHub` during contract creation and assign `driverId` to its return value.

#### **[L-02] SplitsSet Event Emitted When Splits Are Not Modified**
##### **Context**:
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L215

##### **Description:**
`_setSplits` does not revert if `newSplitsHash` equals `state.splitsHash` so the `SplitsSet` event is emitted if the function is called but splits are not modified. This could cause confusion and issues for off-chain indexers.

##### **Recommendation**:
Move the event emission to within the `if` block such that it only occurs when splits are actually modified.

#### **[L-03] Off By One Array Allocation**
##### **Context**:
https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L182-L185

##### **Description:**
The array in `nextSqueezed` is allocated `2 ** 32` elements but this is one more than the maximum number of drips configurations in a given cycle.

##### **Recommendation**:
Allocate `type(uint32).max` i.e. `(2 ** 32) - 1`.

---

### **Non-Critical Issues**

#### **[NC-01] Duplicated & Unused Constants**
##### **Context**:
https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L58

https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L108-L110

##### **Description**:
Constants are duplicated between `Drips.sol` and `DripsHub.sol`, increasing contract bytecode. Given `DripsHub` inherits from `Drips` it is not necessary to include these duplicates. Additionally, `AMT_PER_SEC_EXTRA_DECIMALS` is not used anywhere in `DripsHub` and `_AMT_PER_SEC_EXTRA_DECIMALS` is not used anywhere is `Drips`.

##### **Recommendation**:
Rename `Drips.sol` constants to remove `_` prefix and reference these where required in `DripsHub.sol` rather than duplicating. Additionally, either remove `_AMT_PER_SEC_EXTRA_DECIMALS` or use it in the calculation of `_AMT_PER_SEC_MULTIPLIER`.

#### **[NC-02] Combine Sorted & Duplicate Splits Receiver Checks**
##### **Context**:
https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L237-L240

##### **Description**:
Splits receivers cannot be duplicated within a configuration but by definition requiring `prevUserId < userId` means that subsequent `userId`s cannot be equal to `prevUserId`.

##### **Recommendation**:
Remove line 238: `require(prevUserId != userId, "Duplicate splits receivers");` as this case will be handled by line 239 `require(prevUserId < userId, "Splits receivers not sorted by user ID");`.

#### **[NC-03] Remove Unused Imports**
##### **Context**:
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L5

https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L4 (DripsConfigImpl)

https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L4 (DripsHistory)

##### **Description:**
`DripsHistory` and `DripsConfigImpl` are imported but not used.

##### **Recommendation**:
Remove unused imports.

#### **[NC-04] Fix NatSpec Typo**
##### **Context**:
https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L18

##### **Recommendation**:
Correct 'know' to 'known'.