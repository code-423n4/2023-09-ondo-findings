
# SharesBurnt event is incorrect
In _burnShares, `preRebaseTokenAmount` and `postRebaseTokenAmount` get their value from `getRUSDYByShares(_sharesAmount)` before and after burning shares (`totalShares` and `shares[account]` are updated).

As `getRUSDYByShares` doesn't use any of the updated value and `sharesAmount` is not modified, then `preRebaseTokenAmount` and `postRebaseTokenAmount` will always be equal.

## Code fixed
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L586-L592

```solidity
    uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

    totalShares -= _sharesAmount;

    shares[_account] = accountShares - _sharesAmount;

    uint256 postRebaseTokenAmount = preRebaseTokenAmount;
```