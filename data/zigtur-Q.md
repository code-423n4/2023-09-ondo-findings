# Unused code should be deleted and not commented out

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/MintRateLimiter.sol#L56-L64

MintRateLimiter.sol constructor has some commented out code.

This code should be deleted if not used in production. Here is the fixed constructor:
```solidity
  constructor(uint256 _mintResetDuration, uint256 _instantMintLimit) {
    resetMintDuration = _mintResetDuration; // can be zero for per-block limit
    mintLimit = _instantMintLimit; // can be zero to disable minting

    lastResetMintTime = block.timestamp;
  }
```
