## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | The contracts has used outdated version of openzeppelin library | contracts |
| [L&#x2011;02] | stale or misleading comment in `rUSDYFactory.sol` | 1 |
| [L&#x2011;03] | Multicall being payable is seriously dangerous | 1 |
| [L&#x2011;04] | Calls inside loops that may address DoS | 1 |

### [L&#x2011;01]  The contracts has used outdated version of openzeppelin library
From package.json, It is confirmed that the contracts has used `"@openzeppelin/contracts": "^4.8.3",` which is an old version and has security issues. While using external libraries, Ensure it is of latest version.

### Recommended Mitigation steps
Update the openzeppelin version to v4.9.3

### [L&#x2011;02]  stale or misleading comment in `rUSDYFactory.sol`
In `rUSDYFactory.sol`, There is misleading or stale comment. When asked about it sponsor made it stale Natspec.

```
          ii) Revoke the `MINTER_ROLE`, `PAUSER_ROLE` & `DEFAULT_ADMIN_ROLE` from address(this).
```

### Recommended Mitigation steps
Remove the stale Natspec which are no longer needed in contract to prevent misleading.

### [L&#x2011;03]  `multiexcall()` being payable is seriously dangerous
In `rUSDYFactory.sol`, `multiexcall()` allows for arbitrary batched calls but it is made payable which will cause issues since `address(exCallData[i].target).call` is in a loop, it is not advisable to add payable to the existing call(). This is because msg.value may be used multiple times. 

There is [1 instance](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L126-L137) of this issue:

```Solidity
File: contracts/usdy/rUSDYFactory.sol

  function multiexcall(
    ExCallData[] calldata exCallData
  ) external payable override onlyGuardian returns (bytes[] memory results) {
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

### Recommended Mitigation steps
Current implementation is not recommended, Howeve [this](https://github.com/Uniswap/v3-periphery/issues/52) discussion is helpful to mitigate the issue.

### [L&#x2011;04]  Calls inside loops that may address DoS
In `rUSDYFactory.sol`, `multiexcall()` function,
Calls to external contracts inside a loop are dangerous because it could lead to DoS if one of the calls reverts or execution runs out of gas. Such issue also introduces chance of problems with the gas limits.

**Per SWC-113:**
External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call.

Reference link- https://swcregistry.io/docs/SWC-113

There is [1 instance](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L126-L137) of this issue:

```Solidity
File: contracts/usdy/rUSDYFactory.sol

  function multiexcall(
    ExCallData[] calldata exCallData
  ) external payable override onlyGuardian returns (bytes[] memory results) {
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

### Recommended Mitigation Steps
1) Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop
2) Always assume that external calls can fail
3) Implement the contract logic to handle failed calls