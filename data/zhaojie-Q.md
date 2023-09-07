## DestinationBridge.approve without checking txnHash exists


DestinationBridge.approve without checking txnHash exists leads to skip _checkThresholdMet permission check.

This does not cause any problems at present, but errors may be introduced during the subsequent upgrade. Therefore, advised to check whether txnHash exists.

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
    //@audit when txnHash not exists t.approvers was added
    t.approvers.push(msg.sender);
  }
```
```

  function _checkThresholdMet(bytes32 txnHash) internal view returns (bool) {
    TxnThreshold memory t = txnToThresholdSet[txnHash];
    //@audit t.numberOfApprovalsNeeded = 0 _checkThresholdMet return true
    if (t.numberOfApprovalsNeeded <= t.approvers.length) {
      return true;
    } else {
      return false;
    }
  }

  ```

  But the approver does not mint the token when it approves a non-existent txnHash because _checkAndUpdateInstantMintLimit will check whether the amount is greater than zero, but it is still has risk, suggest check whether txnHash exist when the approve.

  ```

  function approve(bytes32 txnHash) external {
    if (!approvers[msg.sender]) {
      revert NotApprover();
    }
    _approve(txnHash);
    _mintIfThresholdMet(txnHash);
  }
  
  function _mintIfThresholdMet(bytes32 txnHash) internal {
    bool thresholdMet = _checkThresholdMet(txnHash);
    Transaction memory txn = txnHashToTransaction[txnHash];
    if (thresholdMet) {
      _checkAndUpdateInstantMintLimit(txn.amount);
      if (!ALLOWLIST.isAllowed(txn.sender)) {
        ALLOWLIST.setAccountStatus(
          txn.sender,
          ALLOWLIST.getValidTermIndexes()[0],
          true
        );
      }
      TOKEN.mint(txn.sender, txn.amount);
      delete txnHashToTransaction[txnHash];
      emit BridgeCompleted(txn.sender, txn.amount);
    }
  }
  ```