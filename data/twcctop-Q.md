1: 

wrap:error  emit message 

```solidity 
function wrap(uint256 _USDYAmount) external whenNotPaused {
    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
      _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR); 
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
@>    emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));//:qa why transfer from  0
    emit TransferShares(address(0), msg.sender, _USDYAmount);
  }
```
shouldn't transfer from zero,should be 
```solidity 
 emit Transfer(msg.sender,address(this) , getRUSDYByShares(_USDYAmount));
```



2.
_burnShares:error  emit message 
https://github.com/code-423n4/2023-09-ondo/blob/3362e1252f3a54943e2517460e5a7988388bc821/contracts/usdy/rUSDY.sol#L575-L610
```solidity

  function _burnShares(
    address _account,
    uint256 _sharesAmount
  ) internal whenNotPaused returns (uint256) {
    require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

    _beforeTokenTransfer(_account, address(0), _sharesAmount);

    uint256 accountShares = shares[_account];
    require(_sharesAmount <= accountShares, "BURN_AMOUNT_EXCEEDS_BALANCE");

@>    uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

    totalShares -= _sharesAmount;

    shares[_account] = accountShares - _sharesAmount;

@>    uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

    emit SharesBurnt(
      _account,
      preRebaseTokenAmount,
      postRebaseTokenAmount,
      _sharesAmount
    );

    return totalShares;

    // Notice: we're not emitting a Transfer event to the zero address here since shares burn
    // works by redistributing the amount of tokens corresponding to the burned shares between
    // all other token holders. The total supply of the token doesn't change as the result.
    // This is equivalent to performing a send from `address` to each other token holder address,
    // but we cannot reflect this as it would require sending an unbounded number of events.

    // We're emitting `SharesBurnt` event to provide an explicit rebase log record nonetheless.
  }

```
notice that `uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);`  and `uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);` are always same value ,because there is no change in _sharesAmount.
So the emit message preRebaseTokenAmount and postRebaseTokenAmount will always have same value