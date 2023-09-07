# Low issues

## [L-01] Elapsed time should be % day == 0
### Impact 
When setting a new range, the elapsed time between the start and end of the period should be % DAY == 0 to minimize precision errors in the calculation of elapsed days.

### Recommended Mitigation Steps
Add an additional check for `(lastRange.end - endTimestamp) % day == 0` in the `setRange` function.

## [L-02] Incorrect events
### Impact 
When `Transfer` and `TransferShares` events are emitted, the wrong value is set. The USDY amount should be multiplied by `BPS_DENOMINATOR`.

### Recommended Mitigation Steps
Make the following changes:

```solidity
emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR));

emit TransferShares(address(0), msg.sender, _USDYAmount * BPS_DENOMINATOR);
```

## [L-03] Missing check for sharesAmount >= BPS_DENOMINATOR in transferShares
In the `transferShares` function, a check for `sharesAmount >= BPS_DENOMINATOR` is missing,it should be similar to `unwrap` function. Small share amounts may result in precision errors.


## [L-04] setThresholds should not accept empty arrays
### Impact 
If the `setThresholds` function receives empty arrays for `amounts` and `numOfApprovers`, it will delete `chainToThresholds[srcChain]` and the `_attachThreshold` function will fail.

### Recommended Mitigation Steps
Add an additional check for that:

```
f (amounts.length == 0) {
      revert ArrayLengthIsZero();
    }
```

## [L-05] if numOfApprovers[i] <= 2 than tx will be approved everytime
### Impact 
If `numOfApprovers[i] <= 2`, every transaction for this amount will be approved only for one approver.

### Recommended Mitigation Steps
Add an additional check for that:
```
if(numOfApprovers[i] <= 2){
    revert SmallNumOfApprovers
}
```

## [L-06] if numOfApprovers[i] <= 2 than tx will be approved everytime
### Impact 
If `numOfApprovers[i] <= 2`, every transaction for this amount will be approved only for one approver.

### Recommended Mitigation Steps
Add an additional check for that:
```
if(numOfApprovers[i] <= 2){
    revert SmallNumOfApprovers
}
```


## [L-07] Next threshold should be strictly more thna previous
### Impact 
The next threshold should be strictly greater than the previous one. Currently, it is possible for two thresholds for different amounts to be equal.

### Recommended Mitigation Steps
Make the following changes:
```diff
-if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
+if (chainToThresholds[srcChain][i - 1].amount >= amounts[i]) {
}
```

## [L-08] Implement the same ordering restriction for amounts as for thresholds
### Impact 
Implement the same ordering restriction for amounts as for thresholds. Amounts should always be in ascending order.


# Non-critical issues

## [NC-01] multiexcall function does not need to return results
The `multiexcall` function does not need to return `results` because if some of the calls fail, the whole transaction will revert. The result will always be an array of booleans, all equal to true.

## [NC-02] msg.value should be equal to the sum of exCallData[i].value
It should be checked whether `msg.value` is equal to the sum of `exCallData[i].value ` in the multiexcall function. If `msg.value` is greater, any excess ether will be stuck in the contract.

## [NC-03] t.approvers.length > 0 check can be removed
The` t.approvers.length > 0` check in the `_approve` function can be removed because it will have the same behavior. The for loop condition `i < t.approvers.length` already covers the same condition.

## [NC-04] Owner should not remove Axelar Relayer as approver
Owner should not be possible to remove Axelar Relayer as approver. 

```
  function removeApprover(address approver) external onlyOwner {
    delete approvers[approver];
    //@audit nc: shoulde not be able to remove Axelar Relayer
    emit ApproverRemoved(approver);
  }
```