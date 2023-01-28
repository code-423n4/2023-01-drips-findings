## [NC-1] `DripsHub`: Consdier using a bigger `uint` than `uint32` for `DripsHubStorage.nextDriverId`

Currently, with a `registerDriver()` using a max gas of `48757` and median gas of `24857`, it'll take about 1m~2m ETH to cause a loop around and reuse of `driverId`s (Assuming 10gwei/gas)

![](https://i.imgur.com/PlMixnL.png)

Maximum amount of ETH Required to loop driverId at 1gwei/gas

![](https://i.imgur.com/nqAuZhK.png)
![](https://i.imgur.com/wZZcvhG.png)

### Recommendation

Consdier using a bigger `uint` than `uint32` for `DripsHubStorage.nextDriverId`. It can be acheived since the `userId` still has room for more bytes of `driverId`.

![](https://i.imgur.com/OBgjeMj.png)
![](https://i.imgur.com/H8DtLxz.png)

### Link To Affected Code

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/DripsHub.sol#L97

https://github.com/code-423n4/2023-01-drips/blob/9fd776b50f4be23ca038b1d0426e63a69c7a511d/src/AddressDriver.sol#L40
