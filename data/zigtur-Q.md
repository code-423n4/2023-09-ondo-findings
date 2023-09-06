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



# SharesBurnt event is incorrect
In _burnShares, `preRebaseTokenAmount` and `postRebaseTokenAmount` get their value from `getRUSDYByShares(_sharesAmount)` before and after burning shares (`totalShares` and `shares[account]` are updated).

As `getRUSDYByShares` doesn't use any of the updated value and `sharesAmount` is not modified, then `preRebaseTokenAmount` and `postRebaseTokenAmount` will always be equal.

So the event will emit two values that will **always** be the same.

## Proof of Concept
Scope: https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L586-L592, https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L172-L177


Test `test_rUSDY_burn()` in `rUSDY_harness.t.sol` shows an example of it (use `-vvvvv` args with forge, and look at `SharesBurnt` log).

## Recommended Mitigation Steps
Event `SharesBurnt` should emit only one of this variable, as the value doesn't change.