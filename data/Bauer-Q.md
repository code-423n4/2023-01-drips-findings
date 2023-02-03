###  No same value input control
```
 function changeAdmin(address newAdmin) public onlyAdmin {
        _changeAdmin(newAdmin);
    }
```
Recommended Mitigation Steps
Add code like this:
``` if (admin == newAdmin revert ADDRESS_SAME();```


### Use Two-Step Transfer Pattern for Access Controls

```
 function changeAdmin(address newAdmin) public onlyAdmin {
        _changeAdmin(newAdmin);
    }

```
Contracts implementing access control's, e.g. owner, should consider implementing a Two-Step Transfer pattern.
Otherwise it's possible that the role mistakenly transfers ownership to the wrong address, resulting in a loss of the role.

