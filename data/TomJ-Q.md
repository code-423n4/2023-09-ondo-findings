## Table of Contents
### Low Risk Issues
- Consider adding checks of _sender and _recipient being different address for _transferShares() 
- BURNER_ROLE cannot burn when contract paused or cannot burn blocked user's share

&ensp;
## Low Risk Issues
### Consider adding checks of _sender and _recipient being different address for _transferShares()  

rUSDY.sol
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L514-L532

#### Issue
Since there is no input validation of _sender and _recipient being different address,
it is possible to send shares within same addresses for transfer(), transferFrom() and transferShares().
It is ideal to add below check in _transferShares() to avoid user from executing meaningless transaction.
```solidity
require(_sender != _recipient, "SAME_ADDRESS");
```

&ensp;
### BURNER_ROLE cannot burn when contract paused or cannot burn blocked user's share

rUSDY.sol
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L626-L655

#### Issue
One of the reason that BURNER_ROLE can burn rUSDY tokens from any account is to be able to burn tokens that are compromised or stolen.
If this actually happens, then it is an emergency and protocol has to react to the incident as soon as possible. However executing 
burn function is not ideal for this situation since BURNER_ROLE has to specify exact amount of compromised/stolen amount as input parameter
(if the amount is greater the function will revert and if its less then not of all compromised/stolen are burned).
So better approach is to pause a contract or add user account that stole the token to blocklist. However this cause another problem 
because BURNER_ROLE cannot burn rUSDY tokens when contract are paused or cannot burn blocked user's share.
Consider changing burn function so that BURNER_ROLE can burn tokens even if contract are paused or burn tokens from blocklist user.

&ensp;
