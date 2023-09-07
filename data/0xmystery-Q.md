## Gas estimate on SourceBridge.burnAndCallAxelar
The UI may have the gas estimate taken care of. Nonetheless, users interacting with this function have no clue what amount of estimated `msg.value` to send along with the call. Apparently, `burnAndCallAxelar()` only checks if `msg.value` is not zero. 

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L72-L74

```solidity
    if (msg.value == 0) {
      revert GasFeeTooLow();
    }
```
While this prevents the latter from calling the function without sending any ether, it does not ensure the amount of ether sent is adequate to cover the gas fees.

Consider implementing the [Axelar gas estimate](https://docs.axelar.dev/dev/general-message-passing/gas-services/pay-gas) to the function logic where possible.

## rUSDY.approve poses a threat to unsavvy users 
The contract already has [`increaseAllowance()`](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L327-L337) and [`decreaseAllowance()`](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L353-L364) implemented as an alternative to [`approve()`](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L276-L279) that can be used as a mitigation to spender frontrunning the revised allowance. Where possible, remove `approve()` since user can always use `increaseAllowance()` for the same intended purpose.

## Missing crucial check in RWADynamicOracle.simulateRange
Consider adding the following check:  

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L119-L128

```diff
    } else {
      Range memory lastRange = ranges[ranges.length - 1];
      uint256 prevClosePrice = derivePrice(lastRange, lastRange.end - 1);
+      // Check that the endTime is greater than the last range's end time
+      if (lastRange.end >= endTime) revert InvalidRange();
      rangeList[length] = Range(
        lastRange.end,
        endTime,
        dailyIR,
        prevClosePrice
      );
    }
    for (uint256 i = 0; i < length + 1; ++i) {
      Range memory range = rangeList[(length) - i];
      if (range.start <= blockTimeStamp) {
        if (range.end <= blockTimeStamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, blockTimeStamp);
        }
      }
    }
```
This will prevent underflow on line 266 below when internally calling `derivePrice()` in the for loop above:

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L262-L274

```solidity
  function derivePrice(
    Range memory currentRange,
    uint256 currentTime
  ) internal pure returns (uint256 price) {
266:    uint256 elapsedDays = (currentTime - currentRange.start) / DAY;
    return
      roundUpTo8(
        _rmul(
          _rpow(currentRange.dailyInterestRate, elapsedDays + 1, ONE),
          currentRange.prevRangeClosePrice
        )
      );
  }
```
## Comment and code mismatch
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L211-L212

```diff
-      // Ensure that the newStart time is less than the end time of the previous range
+      // Ensure that the newStart time is not less than the end time of the previous range
      if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();
```
## Chain reorganization attack
As denoted in the [Moralis academy article](https://academy.moralis.io/blog/what-is-chain-reorganization):

"... If a node receives a new chain that’s longer than its current active chain of blocks, it will do a chain reorg to adopt the new chain, regardless of how long it is."

Depending on the outcome, if it ended up placing the transaction earlier than anticipated, quite a number of the function calls could backfire. For instance, a user calling `rUSDY.transfer()` could end up [transferring more shares](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L451) than anticipated due to a lower `USDY` price entailed.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L388-L392

```solidity
  function getSharesByRUSDY(
    uint256 _rUSDYAmount
  ) public view returns (uint256) {
    return (_rUSDYAmount * 1e18 * BPS_DENOMINATOR) / oracle.getPrice();
  }
```
(Note: On Ethereum this is unlikely but this is meant for contracts going to be deployed on any compatible EVM chain many of which like Polygon, Optimism, Arbitrum are frequently reorganized.)

## Non-linear vs linear representation 
The graphical representation of the price of evolution of USDY over time in [README.md](https://github.com/code-423n4/2023-09-ondo) should be non-linear step wise instead of a continuous straight line plot to better portray the price appreciation via the daily interest set for the month. 

## Smaller periodic USDY appreciation
The interest rate derived for each subsequent day based on the interest rate for the month presents kinks that is too abrupt around the end of the day and the beginning of the next day.

Imagine two users [transferring their rUSDY tokens](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L468) one second apart such that the user A did it at 23:59:59 and user B did it at 00:00:00. For a matter of a second difference, the user A ended up unfavorably transferring a greater amount of shares due to a smaller denominator entailed on the return code line below: 

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L388-L392

```solidity
  function getSharesByRUSDY(
    uint256 _rUSDYAmount
  ) public view returns (uint256) {
return (_rUSDYAmount * 1e18 * BPS_DENOMINATOR) / oracle.getPrice();
  }
```
This could create imbalanced feelings where the user wished he/she did it seconds/minutes/... later.

Consider reducing the daily interest appreciation model to a smaller periodic model to help circumvent the aforesaid situations. 

## Incorrect arguments associated `getRUSDYByShares()` in the function logic of `_burnShares()`
The `preRebaseTokenAmount` and `postRebaseTokenAmount` values will always be identical, which doesn't provide any meaningful information to the user about the impact of the burn operation on the value of their shares.

Consider having the affected codes refactored as follows:

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L575-L601

```diff
  function _burnShares(
    address _account,
    uint256 _sharesAmount
  ) internal whenNotPaused returns (uint256) {
    require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

    _beforeTokenTransfer(_account, address(0), _sharesAmount);

    uint256 accountShares = shares[_account];
    require(_sharesAmount <= accountShares, "BURN_AMOUNT_EXCEEDS_BALANCE");

-    uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);
+    uint256 preRebaseTokenAmount = getRUSDYByShares(accountShares);

    totalShares -= _sharesAmount;

+    uint256 postAccountShares = accountShares - _sharesAmount;
-    shares[_account] = accountShares - _sharesAmount;
+    shares[_account] = postAccountShares;

-    uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);
+    uint256 postRebaseTokenAmount = getRUSDYByShares(postAccountShares);

    emit SharesBurnt(
      _account,
      preRebaseTokenAmount,
      postRebaseTokenAmount,
      _sharesAmount
    );

    return totalShares;
```