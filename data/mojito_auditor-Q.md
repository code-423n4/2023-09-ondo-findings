# Summary

| Id | Title |
| --- | --- |
| L-1 | Function `_mul()` does not work with Solidity 0.8 |
| L-2 | Incorrect naming variable in `unwrap()` function |
| L-3 | Pre and post rebase amount will always be equal |

# L-1. Function `_mul()` does not work with Solidity 0.8

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L405

## Detail
In `RWADynamicOracle` contract, the function `_mul()` implemented overflow check to ensure the result of `x * y` does not overflow. However, this check does not work with Solidity 0.8 because if there is overflow, Solidity 0.8 will automatically revert when doing `z = x * y`. 
```solidity
function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
  // @audit Solidity 0.8 will revert when doing `x * y`
  require(y == 0 || (z = x * y) / y == x); 
}
```
https://docs.soliditylang.org/en/v0.8.19/080-breaking-changes.html#silent-changes-of-the-semantics

## Recommendation
Solidity 0.8 has already checked for overflow so no need for this function. 


# L-2. Incorrect naming variable in `unwrap()` function

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L451

## Detail
In the function `unwrap()`, it calculated the amount of shares from amount of rUSDY. But the result is named `usdyAmount` instead of `sharesAmount`. This is confusing and should be correct to make the code more readable.

```solidity
function unwrap(uint256 _rUSDYAmount) external whenNotPaused {
  require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
  
  // @audit Should be `shareAmount` instead of `usdyAmount`
  uint256 usdyAmount = getSharesByRUSDY(_rUSDYAmount); 
  if (usdyAmount < BPS_DENOMINATOR) revert UnwrapTooSmall();
  _burnShares(msg.sender, usdyAmount);
  usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
  emit TokensBurnt(msg.sender, _rUSDYAmount);
}
```

## Recommendation
Change the variable name to `sharesAmount`.

# L-3. Pre and post rebase amount will always be equal

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L594-L599

## Details
In the function `_burnShares()`, it will emit an event at the end that includes `preRebaseTokenAmount` and `postRebaseTokenAmount`. These amount is calculated in the same way `getRUSDYByShares(_sharesAmount)` before and after burning the shares. However, by looking at the function `getRUSDYByShares()`, it does not take into account the value of `totalShares` so the results before and after burning the shares will be the same.
```solidity
function getRUSDYByShares(uint256 _shares) public view returns (uint256) {
  return (_shares * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
}
```

## Recommendation
Consider removing one of them in the event and rename the other. 
