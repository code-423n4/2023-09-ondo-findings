# [L-1] PausableUpgradeable contract is not initialized

Contract that inherit from PausableUpgradeable should call the `__Pausable_init()` function in their initialize function. This is not done in the rUSDY contract.


https://github.com/code-423n4/2023-09-ondo/blob/b88271d64112234b7a7273cd7f3cea73c350e6a7/contracts/usdy/rUSDY.sol#L128-L131

## Recommended Mitigation Steps:

```diff
    ) internal onlyInitializing {
        __BlocklistClientInitializable_init(blocklist);
        __AllowlistClientInitializable_init(allowlist);
        __SanctionsListClientInitializable_init(sanctionsList);
        __rUSDY_init_unchained(_usdy, guardian, _oracle);
+       __Pausable_init();        
    }
```

# [L-2] Number of approve needed is always minus 1

In the DestinationBridge, the number of approvals needed is always one less than the threshold. This is because there is an automatic approval when `_execute` function is called.

        _approve(txnHash);

https://github.com/code-423n4/2023-09-ondo/blob/b88271d64112234b7a7273cd7f3cea73c350e6a7/contracts/bridge/DestinationBridge.sol#L111

This means that the number of approvals needed is always reduced by one. For example, if the threshold is 2, then the number of approvals needed is 1. If the threshold is 3, then the number of approvals needed is 2.

# [NC-1] Unnecessary overflow checking

In `_rmul` function, the x * y operation is executed in the _mul(x, y) function. The _mul function performs the overflow checking by `require(y == 0 || (z = x * y) / y == x);`. However since solidity 0.8, overflow checking is done by default. Therefore, the overflow checking in _mul function is unnecessary and can be removed.

https://github.com/code-423n4/2023-09-ondo/blob/b88271d64112234b7a7273cd7f3cea73c350e6a7/contracts/rwaOracles/RWADynamicOracle.sol#L405-L407

# [NC-2] State is not cleared for the executed transaction

In the destination bridge, after the transaction is executed, not all the state variables related to the transaction are cleared. There are two state variables:

- txnHashToTransaction is cleared
- txnToThresholdSet is not cleared

Gas can be saved by clearing the txnToThresholdSet state variable.

# [NC-3] Inaccurate variable name

In the unwrap function, the `usdyAmount` is not the amount of USDY to be unwrapped, but the amount of shares to be burned. Therefore, the variable name should be changed to `sharesAmount` instead of `usdyAmount` to avoid confusion. It can mislead the reader to think that the amount of USDY.

https://github.com/code-423n4/2023-09-ondo/blob/b88271d64112234b7a7273cd7f3cea73c350e6a7/contracts/usdy/rUSDY.sol#L451