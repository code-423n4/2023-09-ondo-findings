# Analysis

## Summary

| Count | Topic | 
|:--:|:-------|
| 1 | Introduction |  
| 2 | Audit Approach |  
| 3 | Architecture Recommendation |  
| 4 | Centralization Risk |   
| 5 | Code Review Insights |  
| 6 | Time Spent |

## Introduction

This technical analysis report focuses on the underlying smart contracts of Ondo Finance consisting of the rebasing variant of USDY token and bridging tokens using Axelar Network. This report explores various contract categories, including Rebase token itself, Bridging component and oracle implementation.

## Audit Approach

1. **Initial Scope and Documentation Review**: Thoroughly went through the Contest Readme File and project documentation to understand the protocol's objectives and functionalities.

2. **High-level Architecture Understanding**: Performed an initial architecture review of the codebase by going through all files without going into function details.

3. **Test Environment Setup and Analysis**: Set up a test environment and execute all tests. Additionally, use Static Analyzer tools like Slither to identify potential vulnerabilities.

4. **Comprehensive Code Review**: Conducted a line-by-line code review focusing on understanding code functionalities. 

    * **Understand Codebase Functionalities**: Began by going through the functionalities of the codebase to gain a clear understanding of its operations.
    * **Analyze Value Transfer Functions**: Adopted an attacker's mindset while inspecting value transfer functions, aiming to identify potential vulnerabilities that could be exploited.
    * **Identify Access Control Issues**: Thoroughly examined the codebase for any access control problems that might allow unauthorized users to execute critical functions.
    * **Evaluate Function Execution Order**: Checked random sequence of function executions to ensure the protocol's logic cannot be disrupted by changing the order of calls.
    * **Assess State Variable Handling**: Identified state variables and assess the possibility of breaking assumptions to manipulate them into exploitable states, leading to unintended exploitation of the protocol's functionality.

5. **Report Writing**: Write Report by compiling all the insights I gained throughout the line by line code review.

## Architecture Recommendation

The following architecture improvements and feedback could be considered:

* On destination chain, `approvers` will be required to pay extra gas fees to perform extra operations of Updating mint limit, minting the tokens to sender etc. Here, `approvers` has no incentive to do it and gas fees can be quite high in case the chain is ethereum. So it is recommended to have an additional function which should be called by user to mint the tokens and perform the operation in `_mintIfThresholdMet` function. Or another option is to introduce a fee on transfer of tokens which will be distributed to approvers to compensate for their gas fees.

## Centralization Risk

Role of Admin is very crucial in almost every contract. Here are the Centralization Risk which Users needs to be aware of before using the smart contracts in scope:

* If threshold is set too high, User can potentially never mint their tokens at destination chain.
* Owner can potentially remove all the approvers anytime which can result in User not able to mint tokens at destination chain.
* Owner can rescue any fund on `DestinationBridge` contract.
* Owner can set `mintLimit` as very low value to force DoS on `DestinationBridge` contract.
* `LIST_CONFIGURER_ROLE` can add any address to blocklist, SanctionsList or Allowlist.
* `PAUSER_ROLE` can pause all the operations of `rUSDY` token anytime.

## Code Review Insights

### Bridge Component

**Contracts:**

1. `SourceBridge`: 
2. `DestinationBridge`
3. `MintTimeBasedRateLimiter`

### Insights / Critical Points

* No checks on whether user provided sufficient amount of gas to execute the function at destination chain or not. 
* In case of emergency, if `DestinationBridge` is paused before `SourceBridge`, User can potentially burn their tokens while will fail to mint it back on destination chain.
* Lots of centralization risk in `DestinationBridge` contract which can lead to user not able to mint the token at destination chain.

### rUSDY token Component

**Contracts:**

1. `rUSDYFactory`
2. `rUSDY`
3. `RWADynamicOracle`

### Insights / Critical Points

* 1 Wei of USDY is equivalent to 10_000 rUSDY shares.
* `transferFrom` function reduces `allowances` even if `_sender` is `msg.sender`
* Any address can be added to `Blocklist`, `Allowlist` and `SanctionsList` in  `rUSDY` token by `LIST_CONFIGURER_ROLE` which can impact their ability to use their funds.
* User can wrap or unwrap their `USDY` to or from `rUSDY`.
* Oracle returns the price of USDY which is used to calculate totalSupply.
* If a range is modified, the previous range's closing price will become outdated for all subsequent ranges that are set.

## Time Spent

| Total Number of Hours | 10 |
|:--:|:--:|

### Time spent:
10 hours