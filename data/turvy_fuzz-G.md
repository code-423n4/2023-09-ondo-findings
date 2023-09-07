## the costly _attachThreshold() function can be adjusted to save gas
this:
```solidity
function _attachThreshold(
    uint256 amount,
    bytes32 txnHash,
    string memory srcChain
  ) internal {
    Threshold[] memory thresholds = chainToThresholds[srcChain];
    for (uint256 i = 0; i < thresholds.length; ++i) {
      Threshold memory t = thresholds[i];
      if (amount <= t.amount) {
        txnToThresholdSet[txnHash] = TxnThreshold(
          t.numberOfApprovalsNeeded,
          new address[](0)
        );
        break;
      }
    }
    if (txnToThresholdSet[txnHash].numberOfApprovalsNeeded == 0) {
      revert NoThresholdMatch();
    }
  }
```
can be adjusted to this:
```solidity
function _attachThreshold(
    uint256 amount,
    bytes32 txnHash,
    string memory srcChain
  ) internal {
    Threshold[] memory thresholds = chainToThresholds[srcChain];
    TxnThreshold memory txnThreshold;
    for (uint256 i = 0; i < thresholds.length; ++i) {
      Threshold memory t = thresholds[i];
      if (amount <= t.amount) {
        txnThreshold = TxnThreshold(
        t.numberOfApprovalsNeeded,
        new address[](0)
        );
        break;
      }
    }
    if(txnThreshold.numberOfApprovalsNeeded == 0) {
    revert NoThresholdMatch();
    }
    txnToThresholdSet[txnHash] = txnThreshold;
}
```