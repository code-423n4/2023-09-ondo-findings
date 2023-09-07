# Unknown Dos to the Admin while calling `multiexCall` 
## Summary
While calling the `multiexcall` function inside `SourceBridge.sol` , the admin passes multiple potential calls in the form of array. The function executes multiple calls( or transactions ) in a single call.
But here's the caveat , if one of them fails , the entire set of transactions fail.

```solidity

function multiexcall(
    ExCallData[] calldata exCallData
  ) external payable override onlyOwner returns (bytes[] memory results) {
    results = new bytes[](exCallData.length);
    for (uint256 i = 0; i < exCallData.length; ++i) {
      (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
->      require(success, "Call Failed");
      results[i] = ret;
    }
  }
```

The `require` statement at the end checks if the transaction has succeeded.
If not, it reverts.

Unfortunately, the admin has no way of knowing which transaction failed!
Let's say the admin has 10 transactions , he can't know which transaction was a black sheep to remove and execute the function again.

Situation gets worse when the admin has calls in the factor 100s.

# Recommendation
instead of require statement , try to have some offchain mechanism in conjuction with onchain (emitting events of reverting transaction data to try again with) or just an array of pending/unsuccessful transactions and rest should be proceeded if they all are independent.

