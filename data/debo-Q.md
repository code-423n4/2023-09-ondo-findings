## [L-01] Unsafe erc20 operations
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::324 => IRWALike(_token).transfer(owner(), balance);
2023-09-ondo/contracts/usdy/rUSDY.sol::437 => usdy.transferFrom(msg.sender, address(this), _USDYAmount);
2023-09-ondo/contracts/usdy/rUSDY.sol::454 => usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
2023-09-ondo/contracts/usdy/rUSDY.sol::680 => usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);
```