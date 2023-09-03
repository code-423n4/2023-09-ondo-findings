a) DestinationBridge.sol
   The constructor should validate for valid ondo approver and owner addresses. They could be passed zero addresses.

b) DestinationBridge.sol
   addApprover and removeApprover parameters should be checked for non-zero address.

c) DestinationBridge.sol
   addChainSupport srcContractAddress should be checked for non-zero address

d) DestinationBridge.sol
   Incase AxelarExecutable contract calls _execute twice, the destination bridge contract does not take 
  the necessary precaution to prevent it. The transaction will revert as _approve will fail.