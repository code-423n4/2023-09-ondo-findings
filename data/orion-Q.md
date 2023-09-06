usage of transfer and transferFrom without check of returned bool, the usdy token if ever upgraded to another ERC20 that returns false for transfer failure of tokens some of Ondo's function will works in unintended way,

consider the wrapping function at : https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L434

it uses usdy.transferFrom without check, if usdy.transferFrom returns false on transfer failures, users will get the shares minted without transfering any tokens

Consider using safetransferFrom, or 

```
(,bool s) = usdy.transferFrom(msg.sender, address(this), _USDYAmount);
require(s,"failed");
```
