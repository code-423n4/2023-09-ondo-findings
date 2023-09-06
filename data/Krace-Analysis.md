# Auther: Krace


# Approach

- Read the Readme.md to get an initial understanding of the project.
  - USDY. Maintain three lists: `allowlist` `blocklist` `sanctionsList`. The redemption price of USDY appreciates as time progresses.
  - rUSDY. Anyone who holds USDY could wrap USDY tokens and receive an amount of rUSDY. `RWADynamicOracle.sol` is utilized to determine the current price of USDY. One rUSDY is one dollar. 
  - RWADynamicRateOracle. Post price evolution for USDY on chain.
  - SourceBridge. 
  - DestinationBridge. Handle calls from the Axelar Gateway
- Clone code and set up the test environment 
  - forge test -vvv
- Read the bot-report
- Analyzing the smart contract's logic 
  - Focus on Common Vulnerability Patterns: reentrancy attacks, integer overflow/underflow, and unauthorized access to sensitive functions or data.
  - Examined the use of external dependencies, ensuring that they were appropriately implemented and that their security implications were understood and accounted for.
  - Reviewed the contract's permissions and access control mechanisms, verifying that only authorized users had access to critical functions and data.
  - Simulated various scenarios, such as edge cases and unexpected inputs, to assess how the contract handled exceptional situations.


# Codebase quality analysis

## [rUSDYFactory.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol)
### Analysis
`rUSDYFactory` is used to deploy an upgradable instance of rUSDY using the ProxyAdmin mode.
`guardian` of `rUSDYFactory` could execute arbitrary code.

### Problem
`guardian` has the ability to execute arbitrary code, which poses a risk to the contract.



## [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol)
### Analysis
Interest-bearing ERC20-like token. The balance of `rUSDY` is calculated based on the price of `USDY` and the share.
Anyone could call `wrap` to save `USDY` and get `rUSDY` back. When the price of `USDY` changes, your balance of `rUSDY` will also change, but you still can get the same `USDY` you deposited.

### Problem
`guardian` has all the permissions, can control the oracle, and can manipulate the price of USDY, which is highly dangerous.
Rug Pull: `guardian` has the ability to `burn` `rUSDY` from any user and then obtain the corresponding `USDY`. 



## [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol)
### Analysis
`RWADynamicOracle` is responsible for providing the current price of `USDY`. `ADMIN` can push a new `Range` or modify the existing `Range`.


## [SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol)
### Analysis
Users can pay gas to burn tokens on the source Chain and call AxelarGateway contract to mint tokens on the destination chain.

### Problem
The owner could set the `contractAddress` by `setDestinationChainContractAddress`, if the owner sets a malicious contract, users' funding could be at risk.


## [DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol)
### Analysis
The Destination Brgdge contract coopreates with Source Bridge. Once a transaction has gotten enough approvers, it could be executed to mint token for `txn.sender`, but, the MintedAmount cannot exceed a limit during a period of time.




# Centralization risks
In `rUSDY`, the `guardian` seems to have too much power. It can even directly burn any user's shares and obtain the corresponding amount of `USDY`. This makes it susceptible to rug pull attacks. It is likely to make users suspicious of the contract, and this overly centralized design should be optimized.
`SourceBridge` also gives Owenr too much power to execute arbitrary code.


# Mechanism review
There are three main parts in this project:
1. wrap USDY to get rUSDY, the balance of rUSDY is calculated based on the current price of USDY
2. the RWADynamicRateOracle provides current price of USDY based on the Range calculation.
3. the bridge allows users to burn USDY or other RWA tokens on the source chain and get new tokens on the destination chain
In conclusion, the added code implements three complete functions, aligning with the documentation's design specifications.


# Systemic risks
The potential risk is the Rug Pull attack, the owner of the contract seems to be too powerful, and can withdraw any account's funds.


# Time spent
20 hours

### Time spent:
20 hours