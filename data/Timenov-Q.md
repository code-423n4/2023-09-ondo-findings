## Summary
[I-1] Contract missing `License-Identifier`.
[I-2] NatSpec does not specify that a function can be called only by particular role.

### [I-1] Contract missing `License-Identifier`.
The `SourceBridge` contract lacks `License-Identifier`.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol

#### Recommended Mitigation Steps
Add `License-Identifier`.

### [I-2] NatSpec does not specify that a function can be called only by particular role.
NatSpec should specify that a function can be called only by users with a particular role. However this is not true in most of the functions that have access modifiers.

#### Recommended Mitigation Steps
Add who can call the function in it's NatSpec.