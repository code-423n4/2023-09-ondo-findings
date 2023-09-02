## Low Risk Issues

### [L-01] Wrong Parameter emitted in the `wrap` function

In the `wrap` function from `rUSDY.sol`, the `_USDYAmount` is a parameter for the function, and the `_USDYAmount` parameter is converted to shares by multiplying the amount by the `BPS_DENOMINATOR`. The issue is from the parameters of the event emitted:

```javascript
File: usdy/rUSDY.sol

438    emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
```

The `Transfer` event uses `getRUSDYByShares(_USDYAmount)`. The function `getRUSDYByShares` takes in the amount in shares and converts it to the equivalent balance of `rUSDY` tokens.

The issue is that `_USDYAmount` is NOT the shares amount and would produce an inaccurate equivalent balance, instead, it should be `getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR)`.

If the results of the events emitted are used, this can lead to unintended behaviours.

