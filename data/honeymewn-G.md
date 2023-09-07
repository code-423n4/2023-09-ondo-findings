Apologies for confusion I was getting an error when I tried to submit QA report (low / non-critical). It seems they're not editable.

1. Missing zero address check in SourceBridge
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L125

should add require(contractAdress != 0);

2. Potential version mismatch between Source and Destination Bridges
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L27
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L48

Consider moving VERSION and other shared constants to a separate file

3. Number of approvals should increase with increasing amounts in DestinationBridge

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L255-L279

there's a check for increasing amounts (oddly implemented). should add another check for number of approvals

Additionally make sure that numOfApprovals > 0

4.  Make sure that arrays are not empty in setThreshold
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L257-L262

amounts and numOfApprovers should be non empty. otherwise the function doesn't make sense

5. 