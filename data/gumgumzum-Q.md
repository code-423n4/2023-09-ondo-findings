## Minor precision loss in `rUSDY@unwrap`
Very small quantities of shares (up to `BPS_DENOMINATOR - 1`) can be burned.

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L453C33-L453C33

### Test 
```solidity
  function testPrecisionLoss() public dealAndWrapAlice {
    _setOraclePrice(1.05e18);
    
    uint256 rUSDYAmount = 3e18;

    uint256 usdyAmount = rUSDYToken.getSharesByRUSDY(rUSDYAmount);
    uint256 remainder = usdyAmount % rUSDYToken.BPS_DENOMINATOR();
    uint256 sharesOfAliceBefore = rUSDYToken.sharesOf(alice);
    

    vm.startPrank(alice);
    rUSDYToken.unwrap(rUSDYAmount);
    vm.stopPrank();

    assertEq(rUSDYToken.sharesOf(alice), sharesOfAliceBefore - usdyAmount + remainder);
  }
```

## Potentially unnecessary computation in `rUSDY@_burnShares`
The result of `rUSDY@getRUSDYByShares` shouldn't change between https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L586 and 
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L592

## Array access can be simplified in `RWADynamicOracle@overrideRange`

```solidity
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index 03aa7d6..bc1b8d0 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -198,7 +198,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
     if (indexToModify == 0) {
       // If the length of ranges is greater than 1,
       // Ensure that the newEnd time is not greater than the start time of the next range
-      if (rangeLength > 1 && newEnd > ranges[indexToModify + 1].start)
+      if (rangeLength > 1 && newEnd > ranges[1].start)
         revert InvalidRange();
     }
     // Case 2: The range being modified is the last range
```
