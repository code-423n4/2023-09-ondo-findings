1: 

Error Transfer Emit message.  

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