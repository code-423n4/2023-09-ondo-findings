## Conflicting specification to code implementation
```solidity
if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();
```
This above check that newStart is go below end time of the previous range therefore it ensures newStart is greater than it and not less than it. However it conflicts the statement [here](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L211).

Recommendation:
Ensure the code implementation are in complaint with the statement or vice versa