https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L434-L455

the wrap and unwrap functions are used to transfer USDY -> shares(rUSDY) and back.
In contrast with wrap, unwrap has an additional check which reverts if the usdyAmount to unwrap is smaller than BPS_DENOMINATOR. 
This could create an edge case where someone wraps an amount and then tries to unwrap with where it would fail due to the check.