# Ondo Finance - Analysis 

|Head |Details|
|:----------------|:------|
| Approach taken in evaluating the codebase | What is unique? How are the existing patterns used? |
|Codebase quality analysis| Its structure, readability, maintainability, and adherence to best practices|
|Architecture recommendations| The design of the protocol|
|Centralization risks| power, control, or decision-making authority is concentrated in a single entity|
|Other recommendations| Recommendations for improving the quality of your codebase|
|Conclusion| Final overview of the protocol|
|Time spent| Total time spent during auditing and reviewing the codebase |

## Approach taken in evaluating the codebase

Steps:

- ``Use a static code analysis tool``: Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

    - Insecure coding practices
    - Common vulnerabilities
    - Code that is not compliant with security standards

- ``Read the documentation``: The documentation for Ondo Finance should provide a detailed overview of the protocol and its codebase. This documentation can be used to understand the purpose of the code and to identify potential areas of concern.

- ``Scope the analysis``: Once you have a basic understanding of the protocol and its codebase, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external APIs, or the code that stores sensitive data.

- ``Manually review the code``: Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

   - Unvalidated user input
   - Hardcoded credentials
   - Insecure cryptographic functions
   - Unsafe deserialization

- ``Mark vulnerable code parts with @audit tags``: Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on.

- ``Dig deep into vulnerable code parts and compare with documentations``:  For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- ``Perform a series of tests``: Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:

  - Valid and invalid user input
  - Different types of attacks
  - Different operating systems and hardware platforms

- ``Report any problems``:  If you find any problems with the code, you should report them to the developers of Ondo Finance. The developers will then be able to fix the problems and release a new version of the protocol.

## Codebase quality analysis

### For all In-Scope contracts

- Named imports are not used in any contract.
- Events lack more indexed parameters.
- Structs and errors can use more inline documentation.

### SourceBridge.sol

- The contract has explicit access control modifiers, such as "onlyOwner" to restrict access to function with external calls and to pause or unpause the contract which can limit certain functions with "whenNotPaused" modifier.
- The contract has detailed inline comments explaining the purpose and functionality of the code.
- The contract uses both custom errors with if condition and error messages with require at the same time for different functions. Consider using only one and custom errors with if condition is most recommended.

### DestinationBridge.sol

- The contract has explicit access control modifiers, such as "onlyOwner" to restrict access to function with external calls and to pause or unpause the contract which can limit certain functions with "whenNotPaused" modifier.
- The functionality of the code is well documented with detailed inline comments.

### rUSDY.sol

- The contract is well written with proper inline documentation explain the purpose and functionality of the code.
- The ``_approve()`` function can manually set allowance of spender from owner to 0 before setting it to a newer value allowing to prevent approval front-running.
- The contract has explicit access control modifiers, such as "onlyRole" to restrict access of certain functionalities to selected roles only.

### RWADynamicOracle.sol

- The contract has explicit access control modifiers, such as "onlyRole" to restrict access of certain functionalities to selected roles only and "whenNotPaused" modifier to restrict some functionality when the contract is paused.
- The code has proper inline documentation but ``_rpow()`` function has complex mathematical calculation  but it's functionality and purpose is not documented properly. 

### Other In-Scope contracts
The contracts ``rUSDYFactory.sol`` and ``IRWADynamicOracle.sol`` are well written but ``IRWADynamicOracle.sol`` can use more documentation.

## Architecture recommendations

The protocol is designed in an easy to understand way. Most of the users will interact with ``rUSDY.sol`` contract which is pretty similar to any other ERC20 tokens with additional functionalities like ``wrap`` and ``unwrap`` which are pretty much self-explanatory. Personally, architecture-wise I would rate it on A level.

## Centralization risks

The protocol is well aware of the centralization risk imposed by some of the functions with different access control. Thus, This risk is considered out-of-scope.

## Other recommendations

- Regular code reviews and adherence to best practices
- Conduct external audits by security experts
- Consider open sourcing the contract for community review
- Maintain comprehensive security documentation
- Establish a responsible disclosure policy for vulnerabilities
- Implement continuous monitoring for unusual activity
- Educate users about risks and best practices

## Conclusion

Ondo Finance is a innovative DeFi protocol that introduces rUSDY, an interest bearing stablecoin. The protocol is well-structured and uses a decentralized governance model. The codebase is very solid and test coverage is also very good. Overall this is a very secured and well written protocol.

## Time-spent 

30 Hours

### Time spent:
30 hours