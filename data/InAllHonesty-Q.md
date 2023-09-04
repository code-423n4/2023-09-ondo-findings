### [L01] Function addChainSupport from DestinationBridge.sol should check if the chain to support is already supported to avoid gas waste or other unintended consequences.

```solidity
  function addChainSupport(
    string calldata srcChain,
    string calldata srcContractAddress
  ) external onlyOwner {
+   if (chainToApprovedSender[srcChain] != bytes32(0)) {
+       revert ChainAlreadySupported();
+   }
    chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));
    emit ChainIdSupported(srcChain, srcContractAddress);
  }


// In the Structs, Events, Errors section
+error ChainAlreadySupported();
```

### [QA01] In DestinationBridge.sol there is a comment section that marks `Public Functions`
[Link to the bad comments](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L327-L329)
None of the functions below the comments are public, which is a bit confusing.

