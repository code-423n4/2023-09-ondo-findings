1- Missing events in rescueTokens and zero address check
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L322
```
  function rescueTokens(address _token) external onlyOwner {
    uint256 balance = IRWALike(_token).balanceOf(address(this));
    IRWALike(_token).transfer(owner(), balance);
  }
```

2- Emit should be in next line after uint256 tokensAmount as a solidity good practices
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L421
```
  function transferShares(
    address _recipient,
    uint256 _sharesAmount
  ) public returns (uint256) {
    _transferShares(msg.sender, _recipient, _sharesAmount);
    emit TransferShares(msg.sender, _recipient, _sharesAmount);
    uint256 tokensAmount = getRUSDYByShares(_sharesAmount);
    emit Transfer(msg.sender, _recipient, tokensAmount);
    return tokensAmount;
  }
```

3- Need approve before transferFrom() in  wrap()
  ```
function wrap(uint256 _USDYAmount) external whenNotPaused {
    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
    _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
+   usdy.approve(address(this), _USDYAmount);
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
    emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
    emit TransferShares(address(0), msg.sender, _USDYAmount);
  }
```