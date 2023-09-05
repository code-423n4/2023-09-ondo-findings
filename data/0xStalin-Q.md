## [L-01] Number of approvers can be set to be greather than the existing approvers in the DestinatinationBridge contract

When [setting the tresholds](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L255-L279) in the DestinationBridge, there is no checks to validate if the number of approvers that are been set to the ranges surpasses the existing maximum number of appprovers in the contract.
Setting the number of required approvers greather than the total number of approvers will cause that tx never reaches the approved status until new approvers are added to the DestinationBridge contract.

```solidity
function setThresholds(
  string calldata srcChain,
  uint256[] calldata amounts,
  uint256[] calldata numOfApprovers
) external onlyOwner {
  if (amounts.length != numOfApprovers.length) {
    revert ArrayLengthMismatch();
  }
  delete chainToThresholds[srcChain];
  for (uint256 i = 0; i < amounts.length; ++i) {
    if (i == 0) {
      chainToThresholds[srcChain].push(
        Threshold(amounts[i], numOfApprovers[i])
      );
    } else {
      if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
        revert ThresholdsNotInAscendingOrder();
      }
      //@audit-issue => Not validating if numOfApprovers[i] is greather than the existing number of approvers
      chainToThresholds[srcChain].push(
        Threshold(amounts[i], numOfApprovers[i])
      );
    }
  }
  emit ThresholdSet(srcChain, amounts, numOfApprovers);
}

```

**Fix:**
- Keep track of the total numbers of approvers, and when setting thresholds validate if the number of required approvers is greather than the total approvers, if so, either revert the tx or set the number of required approvers as the total number of approvers, or emit an event to notify off-chain that will be required to add more approvers to the DestinationBridge


## [L-02] When attaching thresholds, if the amount of tokens to be minted exceeds all the thresholds, the tx will be reverted because that amount won't be assigned a number of required approvals!

When [attaching thresholds](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L128-L147) in the DestinationBridge as part of the flow when running the _execute() that is called by the Axelar Gateway, if the amount of tokens to be minted exceeds all the existing thresholds, the numberOfApprovalsNeeded won't be updated, thus, the default value will be 0, which will cause the tx to be reverted.

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
  //@audit-info => If the amount exceeded all the thresholds, the numberOfApprovalsNeeded won't be updated, thus, the tx will be reverted!s
  if (txnToThresholdSet[txnHash].numberOfApprovalsNeeded == 0) {
    revert NoThresholdMatch();
  }
}
```

**Fix:**
Use the last threshold for huge amounts of tokens to minted, instead of reverting the tx, assign a default number of approvers for all the amounts that exceeds a certain amount.
- Example, if the last threshold would be for 1000 tokens to be minted and require 5 approvers, add a new range at the end to prevent the tx if the amount exceeds the 1000 tokens, and set more number of approvers.
If transactions are reverted because the amount exceeds all the thresholds, one of two things will happen, either the admins will need to update the thresholds, or the minting of those tokens will get stuck.