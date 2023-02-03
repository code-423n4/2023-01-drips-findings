[01] ++I/I++ should be unchecked{++I}/unchecked{I++} when it is not possible for them to overflow. This saves 30-40 gas per loop.
There are 16 instances of this issue.

```
File Drips.sol

247:         for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

287:        for (uint32 cycle = fromCycle; cycle < toCycle; cycle++) {

358:        for (uint256 i = 0; i < squeezedNum; i++) {

422:        for (uint256 i = 1; i <= dripsHistory.length && i <= currCycleConfigs; i++) {

450:        for (uint256 i = 0; i < dripsHistory.length; i++) {

479:        for (uint256 idxCap = receivers.length; idx < idxCap;) {

490:        for (; idx < receivers.length; idx++) {

563:        for (uint256 i = 0; i < receivers.length; i++) {

664:            for (uint256 i = 0; i < newReceivers.length; i++) {

745:            for (uint256 i = 0; i < configsLen; i++) {

777:            for (uint256 i = 0; i < receivers.length; i++) {

```

```
File DripsHub.sol

613:         for (uint256 i = 0; i < userMetadata.length; i++) {

```

```
File Splits.sol

127:        for (uint256 i = 0; i < currReceivers.length; i++) {

158:        for (uint256 i = 0; i < currReceivers.length; i++) {

231:        for (uint256 i = 0; i < receivers.length; i++) {

```
```
File Caller.sol

196:         for (uint256 i = 0; i < calls.length; i++) {

```



