# QA report for Ondo Finance contest by Catellatech

## [01] assert is used only in tests 
in the rUSDYFactory contract in the line 100 the devs used the assert instead of require or custom errors.

check the Solidity docs ➡️ https://docs.soliditylang.org/en/v0.8.19/control-structures.html#panic-via-assert-and-error-via-require

## [02] in rUSDYFactory contract and in so many instances the devs doesn't check the imput 
Please ckeck all the input addresses in the functions, constructor. 


## [03] If onlyOwner runs renounceOwnership() the contract become unavailable
The `onlyOwner` authority here is very important for some contracts, but the `Ownable.sol library` imported with the import has the renounceOwnership() feature, in case the owner accidentally triggers this function, the remove functions will not work and the contract will block gas due to arrays, may have a continuous structure that exceeds its limit.

The solution to this is to overide and disable the renounceOwnership() function as implemented in many contracts in his project, it is important to include this code in the contracts.

```solidity
function renounceOwnership() public payable override onlyOwner {
    revert("Cannot renounce ownership");
}
```

## [04] In RWADynamicOracle contract utilize Assembly - should have comments
The Moonwell team makes use of Assembly on RWADynamicOracle contract, since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

### Recommendation
It is essential to clearly and comprehensively document all activities related to critical function `assembly` within the project. By doing so, a more complete and accurate understanding of what is being done is provided, which is important both for auditors and for long-term project maintenance. By following the practices of monitoring and controlling project work, it can be ensured that all assemblies are properly documented in accordance with the project's objectives and goals.

## [05] Some comments don't match what is described
In some contracts, certain things are described that do not match with what follows, for example:

```solidity
DestinationBridge.sol
/*//////////////////////////////////////////////////////////////
                        Public Functions
  //////////////////////////////////////////////////////////////*/

  /**
   * @notice internal function to mint a transaction if it has passed the threshold
   *         for the number of approvers
   *
   * @param txnHash The hash of the transaction we wish to mint
   */
  function _mintIfThresholdMet(bytes32 txnHash) internal {..//}
```

## [06] Inconsistent Use of NatSpec
NatSpec is a boon to all Solidity developers. Not only does it provide a structure for developers to document their code within the code itself, it encourages the practice of documenting code. When future developers read code documented with NatSpec, they are able to increase their capacity to understand, upgrade, and fix code. Without code documented with NatSpec, that capacity is hindered.

The Ondo codebase does have a high level of documentation with NatSpec. However there are numerous instances of functions missing NatSpec.

### Possible Risks
1. Lack of understanding of code without adequate documentation.
2. Difficulty in understanding, upgrading, and fixing code without documentation.

### Recommended Mitigation Steps
- Practice consistent use of NatSpec on all parts of the project codebase.
- Include the need for NatSpec comments for code reviews to pass.

## [07] Old version of OpenZeppelin Contracts used
Using old versions of 3rd-party libraries in the build process can introduce vulnerabilities to the protocol code.

- Latest is 4.9.3 (as of July 28, 2023), version used in project is 4.8.3

### Risks
- Older versions of OpenZeppelin contracts might not have fixes for known security vulnerabilities.
- Older versions might contain features or functionality that have been deprecated in recent versions.
- Newer versions often come with new features, optimizations, and improvements that are not available in older versions.

### Recommendation
Review the latest version of the OpenZeppelin contracts and upgrade.

## [08] Inconsistent coding style
An inconsistent coding style can be found throughout the codebase. Some examples include:
Some of the **parameter names** in certain functions lack the underscores as seen in other cases. To enhance code `readability` and adhere to the Solidity style guide standards, it is recommended to incorporate underscores in the parameter names. **This practice contributes to maintaining a consistent and organized codebase.**


To improve the overall consistency and readability of the codebase, consider adhering to a more consistent coding style by following a specific style guide.

## [09] Crucial information is missing on important functions in all contracts
In the constructor functions it is not specified in the documentation if the admin/roles will be an EOA or a contract. Consider improving the docstrings to reflect the exact intended behaviour, and using `Address.isContract` function from OpenZeppelin’s library to detect if an address is effectively a contract.

## [10] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.
For example:

```Solidity 
mapping(address account => uint256 balance); 
```