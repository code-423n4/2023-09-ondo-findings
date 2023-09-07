# No enforcement on amount of price ranges past the current range
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L151-L171
### Impact
It is possible for more than two price ranges to be set passed the current range. This violates the contract spec.

### POC
As per the contract spec provided in the Code4rena page [here](https://code4rena.com/contests/2023-09-ondo-finance#top), "In RWADynamicOracle, the operator will not add more than 2 ranges passed the current range. Hence, out of gas loop iteration is out of scope". However, in the `setRange()` function in RWADynamicOracle, there are no checks on the amount of ranges created. Hence, it is possible to create more than 2 ranges past the current range by repeatedly calling `setRange()`.

### Recommended Mitigation
Enforce the statement via a condition that checks the number of ranges that fully extend past `block.timestamp`.
