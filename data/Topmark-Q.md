### Issue 1:
Using same name to represent different variables could cause conflict of interest and misrepresentations when dealing with Threshold and TxnThreshold struct in the DestinationBridge.sol contract. Variable Names should be adjusted accordingly.
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L369-L377
```solidity
  struct Threshold {
    uint256 amount;
--->    uint256 numberOfApprovalsNeeded;
  }

  struct TxnThreshold {
--->    uint256 numberOfApprovalsNeeded;
    address[] approvers;
  }
```