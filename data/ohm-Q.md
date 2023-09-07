## ACCOUNT ADDRESS WILL NOT BE ZERO 

```
  function _burnShares(
    address _account,
    uint256 _sharesAmount
  ) internal whenNotPaused returns (uint256) {
     require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

    _beforeTokenTransfer(_account, address(0), _sharesAmount);

    uint256 accountShares = shares[_account];
    require(_sharesAmount <= accountShares, "BURN_AMOUNT_EXCEEDS_BALANCE");

    uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

    totalShares -= _sharesAmount;

    shares[_account] = accountShares - _sharesAmount;

    // ISSUE ----> = postRebaseTokenAmount is not getting the post rusdy tokens
    //Because _sharesAmount is not updating.

    uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);
```
In the above function there is no possibility of becoming the address is 0.So we can avoid the require statement .
```
    require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");
```