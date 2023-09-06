# Summary

## Low Risk Issues
|Id|Title| 
|:--:|:-------|
|L-01| Manually approve allowance to 0 to reduce chance of approval front-running | 


## [L-01] Manually approve allowance to 0 to reduce chance of approval front-running.
### Instance:
1. https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L485-L495

### Description
The EIP-20 token's approve() function creates the potential for an approved spender to spend more than the intended amount. A front running attack can be used, enabling an approved spender to call transferFrom() both before and after the call to approve() is processed. 

###  Recommendations
The ``_approve()`` of ``rUSDY.sol`` should manually approve allowances of spender from owner to 0 to before approving it to any other value. This will prevent approval front-running attacks.

Change From:
```
  function _approve(
    address _owner,
    address _spender,
    uint256 _amount
  ) internal whenNotPaused {
    require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");
    require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");

    allowances[_owner][_spender] = _amount;
    emit Approval(_owner, _spender, _amount);
  }
```
To:
```diff
  function _approve(
    address _owner,
    address _spender,
    uint256 _amount
  ) internal whenNotPaused {
    require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");
    require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");

+   allowances[_owner][_spender] = 0; //manually approve to 0
    allowances[_owner][_spender] = _amount;
    emit Approval(_owner, _spender, _amount);
  }
```


## Non-Critical Issues
|Id|Title
|:--:|:-------|
|NC-01| Use named imports 
|NC-02| Add more indexed parameters in events 
|NC-03| Struct and errors can use more inline documentation 
|NC-04| ``_rpow()`` lacks proper functional documentation


## [NC-01] Use named imports
There are no named imports used in any of the contracts in scope. Named imports should be used throughout the contract for better code readability.

#### Instances:
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L18-L28
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L19-L22
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L18-L20
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L3-L9
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L18-L25

## |NC-02| Add more indexed parameters in events 
At least 3 named indexed parameters can be used in events but at most 2 is used. Using indexed parameters makes it easier and faster to index and search values/parameters for off-chain monitoring systems and interfaces. 

#### Instance:
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L385-L437

## |NC-03| Struct and errors can use more inline documentation 
The structs and errors used throughout the contract lack proper inline documentation. Add proper documentation for better code readability and understanding.

#### Instance:
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L439-L448
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L369-L382

## |NC-04| ``_rpow()`` lacks proper functional documentation
The ``_rpow()`` is a complex function used in calculation to derive the price. But, the functionality and purpose of this function is not properly documented. Add a proper documentation for this function.

#### Instance:
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L342-L345


