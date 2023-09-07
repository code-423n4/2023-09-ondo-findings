## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | `_rmul()` can gas optimized by removing extra lines of code | 1 |
| [G&#x2011;02] |  Use assembly to emit events | 1 |
| [G&#x2011;03] |  Catch arguments as local variables in `simulateRange()` | 1 |
| [G&#x2011;04] |  Catch `value` in `roundUpTo8()` | 1 |
| [G&#x2011;05] |  save gas by using 10e27 instead 10 ** 27 | 1 |
| [G&#x2011;06] |  `_rpow()` and `_rmul()` can be imported instead of hardcoding it | 1 |
| [G&#x2011;07] |  Short the string message in `onlyGuardian` to save 256 bit storage | 1 |

### [G&#x2011;01]  `_rmul()` can gas optimized by removing extra lines of code
`_rmul()` has been used in `derivePrice()`. With current implementation it has added extra bytes resulting in more deployment cost. This can be further gas optimized per recommendation.

There is [1 instance](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L400-L406) of this issue:

```Solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

  function _rmul(uint256 x, uint256 y) internal pure returns (uint256 z) {
    z = _mul(x, y) / ONE;
  }

  function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
    require(y == 0 || (z = x * y) / y == x);
  }
```

### Recommended Mitigation steps

```diff

  function _rmul(uint256 x, uint256 y) internal pure returns (uint256 z) {
-    z = _mul(x, y) / ONE;
+    z = x * y;
+    require(y == 0 || z / y == x);
+       z = z / ONE;
  }

-  function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
-    require(y == 0 || (z = x * y) / y == x);
-  }
```

### [G&#x2011;02]  Use assembly to emit events
Assembly can be used to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

There are 2 instances of this issue:

```Solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

  event RangeSet(
    uint256 indexed index,
    uint256 start,
    uint256 end,
    uint256 dailyInterestRate,
    uint256 prevRangeClosePrice
  );


  event RangeOverriden(
    uint256 indexed index,
    uint256 newStart,
    uint256 newEnd,
    uint256 newDailyInterestRate,
    uint256 newPrevRangeClosePrice
  );
```

### [G&#x2011;03]  Catch arguments as local variables in `simulateRange()`
Caching will replace each Gwarmaccess (100 gas) with a much cheaper stack read.

There is [1 instance](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L104-L139) of this issue:

```Solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

  function simulateRange(
    uint256 blockTimeStamp,
    uint256 dailyIR,
    uint256 endTime,
    uint256 startTime,
    uint256 rangeStartPrice
  ) external view returns (uint256 price) {
```

### Recommended Mitigation steps
Catch function arguments as local variables.

### [G&#x2011;04]  Catch `value` in `roundUpTo8()`
Caching will replace each Gwarmaccess (100 gas) with a much cheaper stack read.

```diff

  function roundUpTo8(uint256 value) internal pure returns (uint256) {
+    uint256 _value = value;
-    uint256 remainder = value % 1e10;
+    uint256 remainder = _value % 1e10;
    if (remainder >= 0.5e10) {
-      value += 1e10;
+      _value += 1e10;
    }
-    value -= remainder;
+    _value -= remainder;
-    return value;
+    _return value;
  }
```

### [G&#x2011;05]  save gas by using 10e27 instead 10 ** 27
The compiler needs to do extra computation while fetching the value 10 ** 27. However this can be optimized by using 10e27.

There is [1 instance](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L343) of this issue:

### Recommended Mitigation steps

```diff
-  uint256 private constant ONE = 10 ** 27;
+  uint256 private constant ONE = 10e27;
```

### [G&#x2011;06]  `_rpow()` and `_rmul()` can be imported instead of hardcoding it
`_rpow()` and `_rmul()` has been taken from makeDao contract, However these functions can be added in library instead of hardcoding it in contract. This will eliminate extra bytes and will make the contract simpler.

### [G&#x2011;07]  Short the string message in `onlyGuardian` to save 256 bit storage
Every character of string message count as 1 byte. Reduce it to save some gas.

```diff

  modifier onlyGuardian() {
-    require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");
+    require(msg.sender == guardian, "rUSDYFactory: Not Guardian");
    _;
  }
```