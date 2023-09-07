## Destination ridge
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L350
It's useless to delete an instance of "txnHashToTransaction" mapping at line 350,because that txnHash won't be used again for it's uniqueness thanks to the nonce value,so no need to delete that element and by doing so the code will be little bit more efficient.