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

Depending on the outcome, if it ended up placing the transaction earlier than anticipated, quite a number of the function calls could backfire. For instance, a user calling `rUSDY.unwrap()` could end up [burning more shares](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L451) than anticipated due to a lower [`USDY` price](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L391) entailed.

(Note: On Ethereum this is unlikely but this is meant for contracts going to be deployed on any compatible EVM chain many of which like Polygon, Optimism, Arbitrum are frequently reorganized.)

 