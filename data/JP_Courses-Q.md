0. QA/LOW: Lack of address(0) checks, relying solely on trusting that the privileged/admin access users will not accidentally pass zero addresses as function parameters.

1. QA: lack of input validation for `_amount`, possible to pass zero value. Possible to send zero rUSDY tokens to another user.

https://github.com/code-423n4/2023-09-ondo/blob/3362e1252f3a54943e2517460e5a7988388bc821/contracts/usdy/rUSDY.sol#L246

Recommendation:
```solidity
  function transfer(address _recipient, uint256 _amount) public returns (bool) {
 ++ require(_amount != 0);
    _transfer(msg.sender, _recipient, _amount);  
    return true;
  }
```

2. QA: seems approve() not internally called, so could use external modifier instead

https://github.com/code-423n4/2023-09-ondo/blob/3362e1252f3a54943e2517460e5a7988388bc821/contracts/usdy/rUSDY.sol#L276

3. QA: unless planning to call this function internally, `public` should be changed to `external` visibility modifier

https://github.com/code-423n4/2023-09-ondo/blob/3362e1252f3a54943e2517460e5a7988388bc821/contracts/usdy/rUSDY.sol#L330

4. LOW: recommend to use safetransferFrom() instead. fail without error message/return value nor revert?

https://github.com/code-423n4/2023-09-ondo/blob/3362e1252f3a54943e2517460e5a7988388bc821/contracts/usdy/rUSDY.sol#L437

