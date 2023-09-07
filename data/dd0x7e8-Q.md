The issue in the `wrap()` function of `rUSDY` contract lies in the event logs for the `Transfer` and `TransferShares` events. Currently, the USDY amount (`_USDYAmount`) is being used as the value for the shares in the event logs, which is incorrect. The accurate number of shares should be `_USDYAmount * BPS_DENOMINATOR`.

```solidity=434
  function wrap(uint256 _USDYAmount) external whenNotPaused {
    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
    _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
    emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
    emit TransferShares(address(0), msg.sender, _USDYAmount);
  }
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L438
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L439

To address this issue, it's recommended to update the event logs to reflect the correct number of shares minted. Here's the corrected code:
```solidity
function wrap(uint256 _USDYAmount) external whenNotPaused {
  require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
  uint256 sharesToMint = _USDYAmount * BPS_DENOMINATOR; // Calculate the correct number of shares
  _mintShares(msg.sender, sharesToMint);
  usdy.transferFrom(msg.sender, address(this), _USDYAmount);
  emit Transfer(address(0), msg.sender, getRUSDYByShares(sharesToMint)); // Use sharesToMint instead of _USDYAmount
  emit TransferShares(address(0), msg.sender, sharesToMint); // Use sharesToMint instead of _USDYAmount
}
```