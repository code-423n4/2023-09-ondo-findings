## Finding Summary 

| ID | Description | Severity |
| - | - | :-: |
| [L-01](#l-01-unnecessary-abundance-of-roles) | Unnecessary abundance of roles in `rUSDY.sol` | Low |
| [L-02](#l-02-unchecked-return-data-when-implementing-multiexcall) | Unchecked return data when implementing multiexcall in `SourceBridge.sol` | Low |

## [L-01] Unnecessary abundance of roles in `rUSDY.sol` 

In `rUSDY.sol` contract, there are plenty of roles used to impement access control system but they are not actually needed as it's the one address that is granted all these roles.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L141-146
```
_grantRole(DEFAULT_ADMIN_ROLE, guardian);
    _grantRole(USDY_MANAGER_ROLE, guardian);
    _grantRole(PAUSER_ROLE, guardian);
    _grantRole(MINTER_ROLE, guardian);
    _grantRole(BURNER_ROLE, guardian);
    _grantRole(LIST_CONFIGURER_ROLE, guardian);
```

This can create additional difficulties when looking at different functions (especially for auditors) as most of them will have different roles.

### Recommendation

Delegate all this roles to `DEFAULT_ADMIN_ROLE`.

## [L-02] Unchecked return data when implementing multiexcall in `SourceBridge.sol` 

When implementing the call using `call()` opcode it's important to check the size of the return data as it's copied to memory. And if it's too big payload returned from `target`, the usage of memory can become expensive and even lead to DoS:

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L165-169
```
   (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
      require(success, "Call Failed");
      results[i] = ret;

```

### Recommendation

Check the return data using assembly.