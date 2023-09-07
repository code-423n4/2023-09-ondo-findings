https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L434-L439

getRUSDYByShares in emit Transfer will return zero because we use the amount not the shares.

recommendation:
emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR));
 