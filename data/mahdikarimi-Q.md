## if no threshold is being set for a specific amount of token and a chain , malicious user can bridge that amount from that chain instantly without needing any approval 

let's break down the issue 
payload is created by encoding VERSION , msg.sender , amount and nonce in source chain so if user use the same amount and same nonce that is not spent by another chain he can have the same payload . 
txnToThresholdSet[txnHash] points to TxnThreshold by hash of a payload as a key . 
txnToThresholdSet[txnHash] is not deleted after _mintIfThresholdMet has been called . 
let's consider the scenario that 1000 tokens need 3 approvals on a chain but no approvals needed for this amount on a another chain , user can bridge 1000 tokens from the first chain then after bridging and get approvals can use the same nonce and tries to bridge 1000 token from other chain since txnToThresholdSet[txnHash] is not being deleted and already contains number of approvals needed plus approvers of the last bridge , and due that no approvals needed is set for this amount and chain , _attachThreshold won't attach a threshold and also won't revert since txnToThresholdSet[txnHash].numberOfApprovalsNeeded is not zero so instantly new tokens are bridged and bypassing threshold . 

## if a TX doesn't meet number of approvals needed the tokens can't be recovered 

there is no mechanism to unlock tokens if approvers don't approve a TX so user losses assets . 
