## Report 1
The need to loop through t.approvers from the code below at every new approver update would take excess gas, using mapping is a better alternative than array, adjusting it in the whole code base now might takes extra effort but considering it might be worth it as mapping is more gas efficient
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L160
```solidity
  function _approve(bytes32 txnHash) internal {
    // Check that the approver has not already approved
    TxnThreshold storage t = txnToThresholdSet[txnHash];
    if (t.approvers.length > 0) {
 ---->     for (uint256 i = 0; i < t.approvers.length; ++i) {
        if (t.approvers[i] == msg.sender) {
          revert AlreadyApproved();
        }
      }
    }
    t.approvers.push(msg.sender);
  }
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L376
```solidity
 struct TxnThreshold {
    uint256 numberOfApprovalsNeeded;
 +++ (address => bool) approvers;
 --- address[] approvers;
  }
```