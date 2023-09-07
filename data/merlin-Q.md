## The burnAndCallAxelar function does not validate the input amount of tokens being burned

The caller has the capability to provide a zero value as the amount when invoking the [SourceBridge.burnAndCallAxelar](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L61) function. Please consider implementing a requirement to disallow zero amount values.

## The rescueTokens function sends tokens to the owner's address rather than the user's address
The `DestinationBridge.rescueTokens` function in `DestinationBridge` is designed to recover tokens that were mistakenly sent to the contract owner's address. This function is implemented incorrectly, and an address should be provided as an argument to specify where these tokens should be sent.
Example:
Alice mistakenly sent a tokens to the `DestinationBridge` contract address. In situations like these, the `DestinationBridge.rescueTokens`  function proves valuable for recovering tokens that were unintentionally sent to a contract. The owner can call the `rescueTokens` function to receive the tokens at their address without the need to specify Alice's address as an argument. Additionally, it's important to use `safeTransferFrom` instead of `transferFrom` for added security.