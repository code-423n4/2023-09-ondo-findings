## Low - Conflicting specification to code implementation
```solidity
if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();
```
This above checks that newStart goes below the end time of the previous range therefore it ensures newStart is greater than it and not less than it. However, it conflicts with the statement [here](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L211).

Recommendation:
Ensure the code implementation is compliant with the statement or vice versa

## Low - Doesn't revert if numOfApprovers[i] == 0

It is assumed that all numOfApprovers of the set thresholds are > 0 and even uses this invariant to determine threshold was found in some function
```
if (txnToThresholdSet[txnHash].numberOfApprovalsNeeded == 0) {
      revert NoThresholdMatch();
    }
```
But doesn't check this when setting thresholds
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L273

Recommendation:
Add a revert check if numOfApprovers[i] == 0 when setting thresholds
