1.Change comparing code to avoid duplicate thresholds.
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L270-L271
The line should be changed as blow:

        if (chainToThresholds[srcChain][i - 1].amount >= amounts[i]) {
          revert ThresholdsNotInAscendingOrder();

2.Add verification of `numOfApprovers` in `DestinationBridge#setThresholds()`.
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L255-L279
The codes should be changed as blow to verify if `numOfApprovers` is reasonable:

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
	        //@audit-info numOfApprovers must not be zero
	        if (numOfApprovers[i] == 0) {
	          revert InvalidThreshold();
	        }
	        chainToThresholds[srcChain].push(
	          Threshold(amounts[i], numOfApprovers[i])
	        );
	      } else {
	        if (chainToThresholds[srcChain][i - 1].amount >= amounts[i]) {
	          revert ThresholdsNotInAscendingOrder();
	        }
	        //@audit-info numOfApprovers must be in ascending order and no duplicate thresholds are allowed.
	        if (chainToThresholds[srcChain][i - 1].numberOfApprovalsNeeded >= numOfApprovers[i]) {
	          revert ThresholdsNotInAscendingOrder();
	        }
	        chainToThresholds[srcChain].push(
	          Threshold(amounts[i], numOfApprovers[i])
	        );
	      }
	    }
	    emit ThresholdSet(srcChain, amounts, numOfApprovers);
	  }

