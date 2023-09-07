# Non-Critical Findings

## [N-01] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## [N-02] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## [N‑03] Use scientific notation (e.g. 1e27) rather than exponentiation (e.g. 10**27)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.

```
uint256 private constant ONE = 10 ** 27;
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L343

## [N-04] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

## [N-05] Use constants for numbers

==In several locations in the code, numbers like 1e36, 100e18, 1e27 are used...==
https://github.com/code-423n4/2021-05-yield-findings/issues/3#issuecomment-852039791

## [N-06] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [N-07] For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
*Recommendation*
`import {contract1 , contract2} from "filename.sol";`

## [N-07] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of
Solidity, which can make the code more insecure and more error-prone.

## [N-08] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

## [N-09] MORE UPDATED VERSION OF SOLIDITY CAN BE USED

Using the more updated version of Solidity can add new features and enhance security. As described in <ins>https://github.com/ethereum/solidity/releases</ins>, Version `0.8.20` is the latest version of Solidity, which includes support for Shanghai. If Optimism does not support `PUSH0` at this moment, Version `0.8.19`, which “contains a fix for a long-standing bug that can result in code that is only used in creation code to also be included in runtime bytecode”, can also be used. To be more secured and future-proofed, please consider using the more updated version of Solidity for the following contracts:

## [N-10] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

## [S-01] You can explain the operation of critical functions in NatSpec with an infographic.