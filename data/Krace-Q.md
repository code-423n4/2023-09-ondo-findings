## [Low] dailyInterestRate maybe wrongly set, causing zero price
`dailyInterestRate` is used to calculate the current price of USDY. Once it is set to zero, the price will become zero.
When admin sets the Range, `dailyInterestRate` should be checked to ensure it's not zero.

```
roundUpTo8(
        _rmul(
          _rpow(currentRange.dailyInterestRate, elapsedDays + 1, ONE),
          currentRange.prevRangeClosePrice
        )
      );
```

## [Info] Wrong comment in DestinationBridge.sol
`_mintIfThresholdMet` is not a public function, maybe you should update your comment.

```
  /*//////////////////////////////////////////////////////////////
                        Public Functions
  //////////////////////////////////////////////////////////////*/

  /**
   * @notice internal function to mint a transaction if it has passed the threshold
   *         for the number of approvers
   *
   * @param txnHash The hash of the transaction we wish to mint
   */
  function _mintIfThresholdMet(bytes32 txnHash) internal {
    bool thresholdMet = _checkThresholdMet(txnHash);
    Transaction memory txn = txnHashToTransaction[txnHash];
    if (thresholdMet) {
      _checkAndUpdateInstantMintLimit(txn.amount);
      if (!ALLOWLIST.isAllowed(txn.sender)) {
        ALLOWLIST.setAccountStatus(
          txn.sender,
          ALLOWLIST.getValidTermIndexes()[0],
          true
        );
      }
      TOKEN.mint(txn.sender, txn.amount);
      delete txnHashToTransaction[txnHash];
      emit BridgeCompleted(txn.sender, txn.amount);
    }
  }
```
