Q1 - Different token name.
Factory emit event with wrong token name, when deploy new token contract.
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L105
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L195

Q2 - Event has wrong amount of transfered shares. 
Function wrap() mint shares with amount - _USDYAmount * BPS_DENOMINATOR. But emit event TransferShares with value of _USDYAmount. Without multiplying by BPS_DENOMINATOR. Correct code:
```
 emit TransferShares(address(0), msg.sender, _USDYAmount * BPS_DENOMINATOR);
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L439

Q3 - Function dont round derived price to the 8th decimal.
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L282C18-L282C18

Q4 - User cant see own token balance, when oracle now has pause mode enabled.
Function balanceOf() call oracle.getPrice(). Function getPrice() has modifier whenNotPaused. 
It is normal that in pause mode the user cannot move their tokens, but viewing the balance should be available. For example, you can display the last price in pause modeю 
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L227

Q5 - Approver can't revoke his vote
There is not opportunity revoke vote for tx in DestinationBridge.sol 
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L197