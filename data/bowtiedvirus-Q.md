# [L1] Lack of contract existence check on call

**Severity:** Low 

**Effected Contract:** SourceBridge.sol, rUSDYFactory.sol

## Summary
The rUSDYFactory and SourceBridge have a multiexcall function, which Allows for arbitrary batched calls to arbitrary address. If the target address of a given call is incorrect, has been destroyed, or is otherwise non-callable, the call will still return true. To fix this, there should be a contract existence check before each call.

```solidity
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

The Solidity documentation has the following warning about using `call`:
The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed.


### Exploit Scenario
Bob, the rUSDY Guardian, calls execute with `calldata[0].target == address(a self-destructed or non-contract address)`. Even though the address no longer points to an existing contract, the operation still succeeds.

## Recommendations
1. Implement a contract existence check before each call in `multiexcall`.

# [L2] Ownership Transfers are a 1-step Process

**Severity:** Low 

**Effected Contract:** SourceBridge.sol, rUSDYFactory.sol, DestinationBridge.sol

## Summary
Contracts using OpenZeppelin's Ownable contract only utilize a single-step `transferOwnership` step.
Consider using Ownable2Step to prevent accidentally locking the owner out.


## Vulnerability Details
Bob, the Owner of SourceBridge, calls `transferOwnership` with an incorrect address. Because this is a one-step process, Bob immediately loses ownership of the Bridge.

## Recommendations
1. Utilize OpenZeppelin's Ownable2Step contract in all contracts that rely on admin usage. This contract is already in the repo, right next to Ownable.sol.

https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable2Step

# [GAS1] Prefer to use custom errors for validation

**Severity:** GAS 

**Effected Contract:** rUSDYFactory.sol, rUSDY.sol

## Summary
Many validations use `require(bool, errMessage)`. However, custom errors are more gas efficient and IMO more readable.

Consider replacing requires with custom errors, which are available as of `sol 0.8.4`.


## Recommendations
1. https://soliditylang.org/blog/2021/04/21/custom-errors/ 
2. https://dev.to/george_k/embracing-custom-errors-in-solidity-55p8 
3. https://medium.com/coinmonks/how-custom-errors-in-solidity-save-gas-3c499aa22745#:~:text=It%20is%20evident%20that%20this%20method%20requires%20way,therefore%20requires%20less%20gas%20at%20time%20of%20deployment

# [QA1] Consider prepending storage variables with `s_` and immutable vars with `i_`. 

**Severity:** QA 

## Summary
The Chainlink style guide has a rule that immutable variables should be prepended with `i_` and storage variables with `s_`. This helps with readability and gas optimization, since it becomes clear which read and writes are expensive.


## Recommendation
1. https://github.com/smartcontractkit/chainlink/blob/develop/contracts/STYLE.md#naming-and-casing

# [QA2] Immutable variables are UPPERCASE but that style is reserved for constants. 

**Severity**: QA 

**Effected Contract:** DestinationBridge.sol#36-43, SourceBridge.sol#20-26

## Summary
In the bridge contracts, immutable variables are named in UPPERCASE. According to the Solidity and Chainlink docs, only constant values should be named in UPPERCASE.


## Recommendation
1. https://docs.soliditylang.org/en/latest/style-guide.html#constants 
2. https://github.com/smartcontractkit/chainlink/blob/develop/contracts/STYLE.md#naming-and-casing