QA report
RWADynamicOracle
1) While the variable names are mostly descriptive, there are places where they could be clearer. For instance, dailyIR could be renamed to dailyInterestRate for clarity.
2) Allow Deletion of Ranges:
```
function deleteRange(uint256 indexToDelete) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(indexToDelete < ranges.length, "Invalid index");
    for (uint256 i = indexToDelete; i < ranges.length - 1; i++) {
        ranges[i] = ranges[i + 1];
    }
    ranges.pop();
}

```

SourceBridge.sol
Add a Function to Remove Supported Chains:
```
function removeDestinationChain(string memory destinationChain) external onlyOwner {
    require(bytes(destChainToContractAddr[destinationChain]).length != 0, "Chain not found");
    delete destChainToContractAddr[destinationChain];
    emit DestinationChainRemoved(destinationChain);
}

```

In the setDestinationChainContractAddress function, consider adding checks to ensure that the provided address is not the zero address.

```
require(contractAddress != address(0), "Invalid address");
```

In the burnAndCallAxelar function, there's a check for msg.value == 0. Consider making the required gas fee a variable or a setting in the contract so that it can be adjusted as gas prices change, rather than hardcoding a check against 0

DestinationBridge.sol
Add a Function to Remove Supported Chains:
```
function removeChainSupport(string memory srcChain) external onlyOwner {
    require(chainToApprovedSender[srcChain] != bytes32(0), "Chain not found");
    delete chainToApprovedSender[srcChain];
    emit ChainSupportRemoved(srcChain);
}

```

In the addChainSupport and setThresholds functions, consider adding checks to ensure that the provided addresses or values are not default values (e.g., address(0) or 0).
```
require(_token != address(0), "Invalid token address");
```