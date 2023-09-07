# Findings Summary

| ID     | Title                                                                                    | Severity      |
| ------ | ---------------------------------------------------------------------------------------- | ------------- |
| [L-01] | setThresholds should prohibit equal amount                                               | Low           |
| [N-01] | Upgradeable contracts should explicitly call the init function of the inherited contract | Non-Critical  |

# Detailed Findings

# [L-01] setThresholds should prohibit equal amount

## Description

```solidity
  function setThresholds(
    string calldata srcChain,
    uint256[] calldata amounts,
    uint256[] calldata numOfApprovers
  ) external onlyOwner {
    if (amounts.length != numOfApprovers.length) {
      revert ArrayLengthMismatch();
    }
    delete chainToThresholds[srcChain];
    for (uint256 i = 0; i < amounts.length; ++i) {
      if (i == 0) {
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      } else {
        if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
          revert ThresholdsNotInAscendingOrder();
        }
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      }
    }
    emit ThresholdSet(srcChain, amounts, numOfApprovers);
  }

  function _attachThreshold(
    uint256 amount,
    bytes32 txnHash,
    string memory srcChain
  ) internal {
    Threshold[] memory thresholds = chainToThresholds[srcChain];
    for (uint256 i = 0; i < thresholds.length; ++i) {
      Threshold memory t = thresholds[i];
      if (amount <= t.amount) {
        txnToThresholdSet[txnHash] = TxnThreshold(
          t.numberOfApprovalsNeeded,
          new address[](0)
        );
        break;
      }
    }
    if (txnToThresholdSet[txnHash].numberOfApprovalsNeeded == 0) {
      revert NoThresholdMatch();
    }
  }
```

At present, `setThresholds` allows the amount to be equal, but in `_attachThreshold`, the equality will not be considered when looking for the threshold, and only the first subscript that meets the conditions will be taken.    
This may not meet the expectations of thresholds. Under strict conditions, the maximum threshold should be taken instead of the first threshold.     

## Recommendations

setThresholds should prohibit equal amount

# [N-01] Upgradeable contracts should explicitly call the init function of the inherited contract

## Description

```solidity
  contract rUSDY is
    Initializable,
    ContextUpgradeable,
    PausableUpgradeable,
    AccessControlEnumerableUpgradeable,
    BlocklistClientUpgradeable,
    AllowlistClientUpgradeable,
    SanctionsListClientUpgradeable,
    IERC20Upgradeable,
    IERC20MetadataUpgradeable

  function initialize(
    address blocklist,
    address allowlist,
    address sanctionsList,
    address _usdy,
    address guardian,
    address _oracle
  ) public virtual initializer {
    __rUSDY_init(blocklist, allowlist, sanctionsList, _usdy, guardian, _oracle);
  }

  function __rUSDY_init(
    address blocklist,
    address allowlist,
    address sanctionsList,
    address _usdy,
    address guardian,
    address _oracle
  ) internal onlyInitializing {
    __BlocklistClientInitializable_init(blocklist);
    __AllowlistClientInitializable_init(allowlist);
    __SanctionsListClientInitializable_init(sanctionsList);
    __rUSDY_init_unchained(_usdy, guardian, _oracle);
  }
```

rUSDY inherits a series of upgradeable contracts from openzeppelin, but does not explicitly call the init function, although currently these init functions do not contain side effects and do not need to be called.
However, there is no guarantee whether logic will be added to the new version of the contract in the future. For compatibility, it is best to explicitly call the init function.

## Recommendations

Explicitly call the init function