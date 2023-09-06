
L-01: _mintIfThresholdMet() A malicious ALLOWLIST can duplicate mint

`_mintIfThresholdMet` does not follow the `Checks Effects Interactions` principle
If ALLOWLIST is a malicious contract, it is possible to repeat the mint token
It is recommended to clear before executing mint

```solidity
  function _mintIfThresholdMet(bytes32 txnHash) internal {
    bool thresholdMet = _checkThresholdMet(txnHash);
    Transaction memory txn = txnHashToTransaction[txnHash];
    if (thresholdMet) {
+     delete txnHashToTransaction[txnHash];    
      _checkAndUpdateInstantMintLimit(txn.amount);
      if (!ALLOWLIST.isAllowed(txn.sender)) {
        ALLOWLIST.setAccountStatus(
          txn.sender,
          ALLOWLIST.getValidTermIndexes()[0],
          true
        );
      }
      TOKEN.mint(txn.sender, txn.amount);
-     delete txnHashToTransaction[txnHash];
      emit BridgeCompleted(txn.sender, txn.amount);
    }
```

L-02: transferFrom() proposes to ignore allowances if `_sender==msg.sender`.

The current `transferFrom()` still requires an allowance if `_sender==msg.sender`.
Some protocols that transfer out tokens use `transferFrom()` instead of `transfer()`.
Suggest adding `_sender==msg.sender` to ignore allowances to accommodate these protocols

L-03: transferFrom() Malicious Allowlist contract can reuse user's allowances 

`transferFrom()` does not follow the `Checks Effects Interactions` principle

Currently it would be the following steps
1. `_transfer()` -> `_beforeTokenTransfer()` -> `_isAllowed()` -> `allowlist.isAllowed`
2. _approve(_sender, msg.sender, currentAllowance - _amount)

This way if the allowlist is maliciously contracted, you can reenter `transferFrom()` to reuse the allowance

Suggestion, modify allowances first, before executing `_transfer()`.

```diff
  function transferFrom(
    address _sender,
    address _recipient,
    uint256 _amount
  ) public returns (bool) {
    uint256 currentAllowance = allowances[_sender][msg.sender];
    require(currentAllowance >= _amount, "TRANSFER_AMOUNT_EXCEEDS_ALLOWANCE");
+   _approve(_sender, msg.sender, currentAllowance - _amount);
    _transfer(_sender, _recipient, _amount);
-   _approve(_sender, msg.sender, currentAllowance - _amount);
    return true;
  }
```


L-04: BURNER_ROLE can't burn off blacklisted user's token

Administrators with `BURNER_ROLE` privileges can execute `burn()` to burn a user's `shares` and transfer `usdy` to themselves.

Currently execute `burn()` -> `_burnShares()` -> `_beforeTokenTransfer()` -> `_isBlocked()` -> `blocklist.isBlocked()`

This way, if the user is blacklisted, the administrator `burn` is unable to execute the

Suggestion

`_beforeTokenTransfer()` no longer checks `_isBlocked/_isSanctioned/_isAllowed` if `msg.sender` has `BURNER_ROLE` privileges.




L-04: unwrap() leaves a small portion of each user in the contract

The current implementation of `unwrap()` is as follows.

```solidity
 function unwrap(uint256 _rUSDYAmount) external whenNotPaused {
    require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
    uint256 usdyAmount = getSharesByRUSDY(_rUSDYAmount);
    if (usdyAmount < BPS_DENOMINATOR) revert UnwrapTooSmall();
@>  _burnShares(msg.sender, usdyAmount);
@>  usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
    emit TokensBurnt(msg.sender, _rUSDYAmount);
  }
```

From the code above we know that `usdy.transfer()` rounds up `usdyAmount`
But `_burnShares()` does not round it up
This results in a mismatch between the user `burn` and the usdy obtained, out of the decimal difference

suggest `_burnShares()` does the rounding


```diff
 function unwrap(uint256 _rUSDYAmount) external whenNotPaused {
    require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
    uint256 usdyAmount = getSharesByRUSDY(_rUSDYAmount);
    if (usdyAmount < BPS_DENOMINATOR) revert UnwrapTooSmall();
-   _burnShares(msg.sender, usdyAmount);
+   _burnShares(msg.sender, usdyAmount / BPS_DENOMINATOR * BPS_DENOMINATOR);
    usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
    emit TokensBurnt(msg.sender, _rUSDYAmount);
  }
```