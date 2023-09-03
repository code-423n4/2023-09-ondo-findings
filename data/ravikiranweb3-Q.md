a) DestinationBridge.sol
   The constructor should validate for valid ondo approver and owner addresses. They could be passed zero addresses.

b) DestinationBridge.sol
   addApprover and removeApprover parameters should be checked for non-zero address.

c)  
   addChainSupport srcContractAddress should be checked for non-zero address