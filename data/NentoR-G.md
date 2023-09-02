File: contracts/bridge/DestinationBridge.sol, line 159.

Unnecessary "if" check for the length of the "approvers" array.
```
 function _approve(bytes32 txnHash) internal {
    // Check that the approver has not already approved
    TxnThreshold storage t = txnToThresholdSet[txnHash];
    if (t.approvers.length > 0) {
      for (uint256 i = 0; i < t.approvers.length; ++i) {
        if (t.approvers[i] == msg.sender) {
          revert AlreadyApproved();
        }
      }
    }
    t.approvers.push(msg.sender);
  }
```

Some gas could be saved by removing the "if" check, like so:
```
 function _approve(bytes32 txnHash) internal {
    // Check that the approver has not already approved
    TxnThreshold storage t = txnToThresholdSet[txnHash];
    for (uint256 i = 0; i < t.approvers.length; ++i) {
      if (t.approvers[i] == msg.sender) {
        revert AlreadyApproved();
      }
    }
    t.approvers.push(msg.sender);
  }
```