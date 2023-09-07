## 1. Audit Approach

|     |     |     |
| --- | --- | --- |
| Step | Task | Details |
| 1   | Run Tests | Tests run successfully |
| 2   | Coverage | ~80% test coverage for contracts in audit scope |
| 3   | Slither | Reviewed Slither results, no vulnerabilities discovered based on output |
| 4   | Surya | Generate graphs to understand the overall project structure. Provided an initial insight to the contract inheritance and function call flow |
| 5   | Solidity Metrics | Generate metrics reports to obtain initial insight on the codebase, noting areas of potential concern |
| 6   | Code Review | Line by line code review |
| 7   | Test Review | Review of each test and it's purpose |

## 2. Mechanism Summary

USDY is a stablecoin previously introduced by Ondo Finance and can be wrapped into `rUSDY`, a new rebasing variant. This uses a newly developed oracle, `RWADynamicOracle` to determine the USDY price. A bridge is also introduced to make the token compatible across multiple chains, using the `SourceBridge` and `DestinationBridge` contracts.

## 3. Centralisation Risks

`USDY` is a centrally managed, upgradable contract. The admin is able to change the code, preventing the contract from working as intended and in the worst case could steal user's funds.

## 4. Quality Analysis

### 4.1 Codebase

The code is well structured and organised into separate contracts, each with a clear purpose. NatSpec comments are well used throughout the codebase. The [automated findings](https://github.com/code-423n4/2023-09-ondo/blob/main/bot-report.md) show that some conventions which would result in saving gas are are ignored and should be taken into consideration, such as using custom errors instead of reverting with a string and using private for constants.

### 4.2 Documentation

The code is well commented with NatSpec comments, which are clear and informative. This could be improved by adding supplementary external documentation with diagrams outlining the workings of the core functionality.

### 4.3 Tests

Tests are clearly structured and cover most of the core functionality but coverage could be better than 80%. Testing could be further improved with the use of fuzz testing, or formal verification to give users the extra assurance that their funds are safe.

## 5. Architecture Improvements

### 5.1 Use an off chain deployment script instead of a factory contract

`rUSDYFactory` is used to deploy `rUSDY` and setup the proxy. This could be done off chain with a deployment script which keeps unnecessary code off chain, saving gas.

### Time spent:
20 hours