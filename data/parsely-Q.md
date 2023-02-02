This is only informational and may be known : 
## 1)

In the DripsHub.sol contract the registerDriver function can be called by any caller and can include a zero address. This has no impact but could fill up the mapping with random data that is never used.
This function below can be pasted into the DripsHub.t.sol contract
```
       function testAnyoneCanRegisterDriver() public {
        //Set up a random address for a user
        address randomUser = address(3);
        address driverAddr = address(0x1234);
        uint32 nextDriverId = dripsHub.nextDriverId();
        assertEq(address(0), dripsHub.driverAddress(nextDriverId), "Invalid unused driver address");
        assertEq(nextDriverId, dripsHub.registerDriver(driverAddr), "Invalid assigned driver ID");
        vm.startPrank(randomUser);
        uint32 nextDriverId1 = dripsHub.nextDriverId();
        assertEq(nextDriverId1, dripsHub.registerDriver(address(0)), "Invalid assigned driver ID");
        
        //Now just fill up the mapping with zero address entries
        for(uint32 i;i<100;i++)
        {
            dripsHub.registerDriver(address(0));
        }
        uint32 nextDriverId2 = dripsHub.nextDriverId();
        console.log(nextDriverId2);
        vm.stopPrank();
        assertEq(nextDriverId2, dripsHub.nextDriverId(), "Invalid next driver ID");
    }
``` 
A check for address(0) could be implemented to mitigate this.

## 2)
In the NFTDriver contract anyone can mint on behalf of any address, this has no impact except it could fill the mapping with random data.
The function below can be pasted into the NFTDriver.t.sol.
```
function testAnyoneCanMint() public  {
        address someuser = address(55);
        vm.startPrank(someuser);
        uint256 tokentest = driver.mint(admin, new UserMetadata[](0));
        uint256 tokentest1 = driver.mint(admin, new UserMetadata[](0));
       //Now check that the admin has 2 new tokens as their balance
        console.log(IERC721(address(driver)).balanceOf(admin));
        console.log(tokentest);
        console.log(tokentest1);
        vm.stopPrank();
    }
```
Perhaps there is no way around this except perhaps to limit the minting of tokens to the ```msg.sender``` address, and/or limit the amount of drivers any one address can register.