# [L-01] There is no way to retrieve the burned funds back on the source chain, if the destination chain transaction cannot be executed for whatever reason

## Impact
If the `DestinationBridge::_execute` function cannot be executed successfully and always reverts, the user funds might be lost forever.


## Proof of Concept

One possible scenario where this issue is possible, is if there is no threshold for the amount that is being bridged.

Let's take the following example: 

1. Alice decides to bridge 1000 `USDY` from Arbitrum to Optimism
2. She calls the Arbitrum SourceBridge's `burnAndCallAxelar` function with `amount` equal to `1000` and `destinationChain` equal to `optimism`
3. The `execute` function on the Optimism DestinationBridge gets executed
4. However, if there is no matching threshold for the amount being bridged, the transaction will always revert in the `_attachThreshold` function (in our example, if the max supported threshold amount is less than 1000), meaning that the funds will be lost forever, or until the DestinationBridge owner comes in and updates the threshold to support the amount sent by Alice

## Tools Used

Manual review

## Recommended Mitigation Steps

There is no straightforward fix for this issue.

One potential way of mitigating it, is to add an expiration timestamp for the given transaction nonce and provide a mechanism for the users to mint back their burned tokens on the source chain, if the transaction on the destination chain has not been executed successfully after the expiration time has passed. However, in order for this to be implemented, there has to be a way to check whether the transaction has been successfully executed or not on the destination chain, from the source chain.

# [N-01] Typos

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L252

```diff
    * @dev This function will remove all previously set thresholds for a given chain
-   *      and will thresholds corresponding to the params of this function. Passing
+   *      and will set thresholds corresponding to the params of this function. Passing
    *      in empty arrays will remove all thresholds for a given chain
    */
```