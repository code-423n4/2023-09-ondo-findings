# Report

## Gas Optimization

|               | Issue | Instances |
| ---------------------- | ------------ | -------------- |
| [GAS-1]   | Cache array length outside of loop     | 5           |
| [GAS-2] | Don't initialize variables with default value         | 8        |
| [GAS-3]    | Functions guaranteed to revert when called by normal users can be marked payable       | 12            |

### [GAS-1] Cache array length outside of loop

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (5)*:

```
File: SourceBridge.sol

    for (uint256 i = 0; i < exCallData.length; ++i)

```

```
File: DesinationBridge.sol

134    for (uint256 i = 0; i < thresholds.length; ++i)

160      for (uint256 i = 0; i < t.approvers.length; ++i) 

264    for (uint256 i = 0; i < amounts.length; ++i)


```

```
File: rUSDYFactory.sol

130    for (uint256 i = 0; i < exCallData.length; ++i)

```

### [GAS-2] Don't initialize variables with default value

*Instances (8)*:

```
File: SourceBridge.sol

    for (uint256 i = 0; i < exCallData.length; ++i)

```

```
File: DesinationBridge.sol

134    for (uint256 i = 0; i < thresholds.length; ++i)

160      for (uint256 i = 0; i < t.approvers.length; ++i) 

264    for (uint256 i = 0; i < amounts.length; ++i)


```

```
File: rUSDYFactory.sol

130    for (uint256 i = 0; i < exCallData.length; ++i)

```

```
File: RWADynamicOracle.sol

77    for (uint256 i = 0; i < length; ++i)

113    for (uint256 i = 0; i < length; ++i)

129    for (uint256 i = 0; i < length + 1; ++i)
```

### [GAS-3] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (12)*:

```
File: SourceBridge.sol

121  function setDestinationChainContractAddress(
    string memory destinationChain,
    address contractAddress
  ) external onlyOwner

136  function pause() external onlyOwner 

145  function unpause() external onlyOwner
```

```
File: DesinationBridge.sol

210  function addApprover(address approver) external onlyOwner

220  function removeApprover(address approver) external onlyOwner

234  function addChainSupport(
    string calldata srcChain,
    string calldata srcContractAddress
  ) external onlyOwner

255  function setThresholds(
    string calldata srcChain,
    uint256[] calldata amounts,
    uint256[] calldata numOfApprovers
  ) external onlyOwner

286  function setMintLimit(uint256 mintLimit) external onlyOwner 

295  function setMintLimitDuration(uint256 mintDuration) external onlyOwner

304  function pause() external onlyOwner

313  function unpause() external onlyOwner

322  function rescueTokens(address _token) external onlyOwner
```

