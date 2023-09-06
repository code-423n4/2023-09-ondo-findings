## [L-01] Unsafe erc20 operations
Description
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::324 => IRWALike(_token).transfer(owner(), balance);
2023-09-ondo/contracts/usdy/rUSDY.sol::437 => usdy.transferFrom(msg.sender, address(this), _USDYAmount);
2023-09-ondo/contracts/usdy/rUSDY.sol::454 => usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
2023-09-ondo/contracts/usdy/rUSDY.sol::680 => usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);
```