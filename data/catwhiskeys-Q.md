# 1) Function with wrong functionality
The `_mul` function wasn't coded well. It should return multiplication of 2 numbers, but the function might return nothing.

Instance:
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L405

PoC
For example, x=1, y=2. In this case, function will not return anything, because it doesn't pass any of the conditions.
y != 0 and y!= x. The `_mul` function provides no value.

Recommended mitigation steps:
According to Company's needs, consider updating this function in this way:
```
function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
	if (y != 0) {
	return z = x * y;
}
```

# 2) Unauthorized users can pause oracle.
Even though function `pauseOracle` is included in the admin's functions, it can be paused by any user.
Apparently, Company doesn't need oracle to be paused by unauthorized users, so consider adding access control checks.

Instance:
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L241

Recommended mitigation steps:
Consider making this function similar to `unpauseOracle`, by making it available only for admin.
```
- function pauseOracle() external onlyRole(PAUSER_ROLE) {
+ function pauseOracle() external onlyRole(DEFAULT_ADMIN_ROLE) {
    _pause();
}
```
