# First Flight #1: PasswordStore

<br />

## Goals

- [x] Check if variables are used.
- [x] Check if password is private.
- [x] Check if password can be correctly updated only by owner.

## Notes

- Other than the found issues, take a look at this finding: [link](https://www.codehawks.com/report/clnuo221v0001l50aomgo4nyn#:~:text=Low%20Risk%20Findings-,L%2D01.%20Initialization%20Timeframe%20Vulnerability,-Submitted%20by%20dianivanov)

## Variables Tree

- address private s_owner;
- string private s_password;

## Findings

- <p>PasswordStore::s_owner is not redefined, thus it can be immutable.</p>

```diff
- address private s_owner;
+ address private immutable s_owner;
```

- <p>PasswordStore::setPassword has no ownership restrictions, thus any user can change the password.</p>

Consider adding a modifier to check if sender is the owner:

```solidity
modifier onlyOwner() {
  if (msg.sender != s_owner) {
      revert PasswordStore__NotOwner();
  }
  _;
}
```

<p>Then we can also add this modifier both to PasswordStore::setPassword and PasswordStore::getPassword:</p>

```diff
-    function setPassword(string memory newPassword) external {
        s_password = newPassword;
        emit SetNetPassword();
    }

-    function getPassword() external view returns (string memory) {
-        if (msg.sender != s_owner) {
-            revert PasswordStore__NotOwner();
-        }
        return s_password;
    }

+    modifier onlyOwner() {
+      if (msg.sender != s_owner) {
+          revert PasswordStore__NotOwner();
+      }
+      _;
+    }

+    function setPassword(string memory newPassword) external onlyOwner {
        s_password = newPassword;
        emit SetNetPassword();
    }

+    function getPassword() external onlyOwner view returns (string memory) {
        return s_password;
    }
```