# TransferShares event emit is not coherent on the field sharesValue

[TransferShares event](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L157) on some instances emits the number of shares, static number with precision 1e18 * 10_000, while on another instances emits the rebasing value of the shares, the balance of the rebasing token, with precision 1e18.

The name of the field, "sharesValue", seems to indicate that the rUSDY rebasing value is to be sent, instead of the term "sharesAmount" used in other parts of the code, to refer to number of shares.

[On the function TransferShares the number of shares is sent](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L421)
[On the function wrap it is sent the shares usdy value](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L439)
[On the internal function _transfer, used on the function transfer and transferFrom, again the number of shares is sent](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L471)

# rUSDYFactory event rUSDYDeployed sends a different name than the deployed rUSDY contract

rUSDYFactory event rUSDYDeployed sends on the name field "Ondo Rebasing U.S. Dollar Yield"; while on the deployed rUSDY contract the hardcoded name() function returns "Rebasing Ondo U.S. Dollar Yield".

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L105
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L194-L196

# DestinationBridge._mintIfThresholdMet function is internal but its located on the Public Functions section of the file

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L327-L353
