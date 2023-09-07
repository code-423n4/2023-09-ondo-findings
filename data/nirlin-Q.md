## NC 1 : Use the uniswap multicall implementation that returns that exact error code, instead of the error code of multicall contract itself
Ondo use the following implementation of multicall in factory and source bridge:

```solidity
  function multiexcall(
    ExCallData[] calldata exCallData
  ) external payable override onlyOwner returns (bytes[] memory results) {
    results = new bytes[](exCallData.length);
    for (uint256 i = 0; i < exCallData.length; ++i) {
      (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
      require(success, "Call Failed");
      results[i] = ret;
    }
  }
 ```

which simply returns the `Call Failed` error on !success, but use the uniswap impementation which returns the exact error code. Implementation is as follow:
https://github.com/Uniswap/v3-periphery/blob/b13f9d9d9868b98b765c4f1d8d7f486b00b22488/contracts/base/Multicall.sol#L11-L28

```solidity
    function multicall(bytes[] calldata data) public payable override returns (bytes[] memory results) {
        results = new bytes[](data.length);
        for (uint256 i = 0; i < data.length; i++) {
            (bool success, bytes memory result) = address(this).delegatecall(data[i]);

            if (!success) {
                // Next 5 lines from https://ethereum.stackexchange.com/a/83577
                if (result.length < 68) revert();
                assembly {
                    result := add(result, 0x04)
                }
                revert(abi.decode(result, (string)));
            }

            results[i] = result;
        }
    }
}
```

## NC 2 : No need to use this much assembly in _rpow
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L345-L398
`_rpow` function us copied from the following repositery,

https://github.com/makerdao/dss/blob/master/src/jug.sol

which is written in very old solidity version 0.6.12 when there was no native way to take power in solidity but now that is not the case.

So as the _rpow represents the following formula given by dev:

result = (prevRangeClose * [(numerator/ ONE) ** (elapsedDays + 1)])

This can be easily done using simple solidity instead use that.


## NC 3:  Use the mul functions copied as it is
`_rmul` and `_mul` function have been copied from following file:

https://github.com/makerdao/dss/blob/master/src/jug.sol

But developers have split it instead for which there is no need to do so, instead use it as or not use it at all. Making changing is susceptible to error

## NC 4 : No need for underflow check in _mul function
The codebase from where the following function is copied  

```solidity
  function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
    require(y == 0 || (z = x * y) / y == x);
  }
```

is written in old solidity version when there were no checks for the underflow or overflow, but with solidity 0.8.0 and later that is not the case. we can see the `_mul` make such simple multiplication a complete headache with those unnecessary checks which are already checked by in solidity version being used which is 0.8.16

## NC 4 :  Every role is granted to guardian, there always should be the separation of concern
Grant the following roles to different addresses:

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L134-L147


```solidity
  function __rUSDY_init_unchained(
    address _usdy,
    address guardian,
    address _oracle
  ) internal onlyInitializing {
    usdy = IUSDY(_usdy);
    oracle = IRWADynamicOracle(_oracle);
    _grantRole(DEFAULT_ADMIN_ROLE, guardian);
    _grantRole(USDY_MANAGER_ROLE, guardian);
    _grantRole(PAUSER_ROLE, guardian);
    _grantRole(MINTER_ROLE, guardian);
    _grantRole(BURNER_ROLE, guardian);
    _grantRole(LIST_CONFIGURER_ROLE, guardian);
  }
```

## NC 5: Unnecessary check in decrease allowance in rUSDY.sol
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L353-L364
```solidity
  function decreaseAllowance(
    address _spender,
    uint256 _subtractedValue
  ) public returns (bool) {
    uint256 currentAllowance = allowances[msg.sender][_spender];
    require(
      currentAllowance >= _subtractedValue,
      "DECREASED_ALLOWANCE_BELOW_ZERO"
    );
    _approve(msg.sender, _spender, currentAllowance - _subtractedValue);
    return true;
  }
```
here no need for the check it will throw underflow check anyways

## Low 1: If the oracle returned zero price ever the protocol is doomed.
oracle is an external factor that cannot be always trusted, and in this case the impact is too much. If the oracle returned the zero price, balances and shares in the `rUSDY.sol` become zero:

```solidity
  function balanceOf(address _account) public view returns (uint256) {
    return (_sharesOf(_account) * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
  }
  ```
  
```solidity
  function totalSupply() public view returns (uint256) {
    return (totalShares * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
  }
```

```solidity
  function getSharesByRUSDY(
    uint256 _rUSDYAmount
  ) public view returns (uint256) {
    return (_rUSDYAmount * 1e18 * BPS_DENOMINATOR) / oracle.getPrice();
  }

  /**
   * @return the amount of rUSDY that corresponds to `_shares` of usdy.
   */
  function getRUSDYByShares(uint256 _shares) public view returns (uint256) {
    return (_shares * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
  }
  ```
  
 
