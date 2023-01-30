# [G-01] Use seperate internal function for checking unique and sorted elements

Splits.sol: [L226](https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L226)

Current implementation checks if the ```i``` is greater than ```0``` on every iteration that is unnecessary, these changes optimizes and saves gas.

```diff
     function _assertSplitsValid(SplitsReceiver[] memory receivers, bytes32 receiversHash) private {
         require(receivers.length <= _MAX_SPLITS_RECEIVERS, "Too many splits receivers");
+        _assertSortedAndUnique(receivers);
         uint64 totalWeight = 0;
-        // slither-disable-next-line uninitialized-local
-        uint256 prevUserId;
         for (uint256 i = 0; i < receivers.length; i++) {
             SplitsReceiver memory receiver = receivers[i];
             uint32 weight = receiver.weight;
             require(weight != 0, "Splits receiver weight is zero");
             totalWeight += weight;
             uint256 userId = receiver.userId;
-            if (i > 0) {
-                require(prevUserId != userId, "Duplicate splits receivers");
-                require(prevUserId < userId, "Splits receivers not sorted by user ID");
-            }
-            prevUserId = userId;
             emit SplitsReceiverSeen(receiversHash, userId, weight);
         }
         require(totalWeight <= _TOTAL_SPLITS_WEIGHT, "Splits weights sum too high");
     }
 
+    function _assertSortedAndUnique(SplitsReceiver[] memory receivers) internal pure {
+        uint256 receiversLength = receivers.length;
+        for(uint256 i = 1; i < receiversLength;) {
+            require(receivers[i - 1].userId != receivers[i].userId, "Duplicate splits receivers");
+            require(receivers[i - 1].userId < receivers[i].userId, "Splits receivers not sorted by user ID");
+
+            unchecked {
+                ++i;
+            }
+        }
+    }
+
```

## Gas report diffs

```
$ forge snapshot --diff
...
testUncollectedFundsAreSplitUsingCurrentConfig() (gas: 53 (0.008%)) 
testSplitSplitsFundsReceivedFromAllSources() (gas: 66 (0.009%)) 
testSplitsConfigurationIsCommonBetweenTokens() (gas: 26 (0.009%)) 
testForwardSplits() (gas: 53 (0.020%)) 
testSimpleSplit() (gas: 33 (0.020%)) 
testSplitRevertsIfInvalidCurrSplitsReceivers() (gas: 33 (0.031%)) 
testSetSplits() (gas: 33 (0.050%)) 
testSetSplits() (gas: 33 (0.052%)) 
testRejectsTooHighTotalWeightSplitsReceivers() (gas: 68 (0.062%)) 
testUserCanSplitToThemselves() (gas: 347 (0.124%)) 
testSplitMultipleReceivers() (gas: 347 (0.132%)) 
testSplittingSplitsAllFundsEvenWhenTheyDoNotDivideEvenly() (gas: 434 (0.165%)) 
testCanSplitAllWhenCollectedDoesNotSplitEvenly() (gas: 434 (0.179%)) 
testCreateSplits() (gas: 434 (0.429%)) 
testRejectsZeroWeightSplitsReceivers() (gas: 66 (0.468%)) 
testLimitsTheTotalSplitsReceiversCount() (gas: 79832 (0.820%)) 
testSplitFundsAddUp(uint256,uint256,uint128,(uint256,uint32)[200],uint256,uint256) (gas: -728421 (-9.149%)) 
testRejectsUnsortedSplitsReceivers() (gas: -2043 (-10.501%)) 
testRejectsDuplicateSplitsReceivers() (gas: -2237 (-12.887%)) 
```
```
Overall gas change: -650409 (-29.960%)
```