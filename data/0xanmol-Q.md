https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L324C15-L324C20

The ```rescueTokens()``` in Destination bridge is not checking the return value of the ERC20 token. Some of the tokens may return false and not checking for the return value can lead to the function passing without the desired outcome.