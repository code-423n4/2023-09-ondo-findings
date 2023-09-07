# Gas Report

## Summary

### Gas Optimizations

|  | Issue | Instances |
| --- | --- | --- |
| [G‑01] | preRebaseTokenAmount & postRebaseTokenAmount are same | 1 |

Note: The table above as well as its gas numbers are created by considering the **automatic findings** which are not included.

---

## Gas Optimizations

### [G‑01]

summary: The preRebaseTokenAmount & postRebaseTokenAmount are same and its not gas efficient.

*There is 1 instance of this issue:*

```solidity
uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);
```

```solidity
uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);
```

Link(s):
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L586
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L592

Recommended mitigation step:
use only one of them.