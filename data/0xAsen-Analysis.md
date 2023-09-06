## 1. Summary
#### 1.1 Description
The main assets in the protocol are USDY and rUSDY which is actually not a real token but it is kept as an internal balance in the rUSDY.sol contract.
`rUSDY` is interest-bearing stablecoin users receive in return for wrapping their USDY. Each 1 rUSDY token is worth 1 dollar and its yield is calculated based on the time you wrapped your USDY and the price of USDY. The yield is implemented by rebasing the supply of rUSDY accordingly when the price of USDY increases.
#### 1.2 Contracts in scope and how to they interact with each other
The contracts in scope were:
SourceBridge.sol - Source chain bridge contract for USDY
DestinationBridge.sol - Destination chain bridge contract for USDY

rUSDY.sol - rebasing USDY token contract
rUSDYFactory.sol - used for deploying rUSDY

RWADynamicOracle.sol - custom oracle used by rUSDY to get the price of USDY and rebase accordingly

Interactions: 
SourceBridge.sol interacts with DestinationBridge.sol through the AxelarGateway enabling users to bridge USDY from one chain to another. 

rUSDYFactory.sol is used to deploy rUSDY.sol and rUSDY.sol interacts with RWADynamicOracle.sol to receive the price of USDY and measure accrued yield by rebasing.

#### 1.3 User flow
For rUSDY.sol:
A user has USDY tokens —> wraps them with the wrap function and receives rUSDY i.e. “shares” in return. The shares amount he receives is the USDYamount * 10_000 —> unwraps them and receives his USDY back including the accrued yield for the time he held rUSDY
A user also has the functionality to transfer and approve rUSDY tokens.

For bridging: 
A user has USDY on Source chain —> calls `burnAndCallAxelar` where he passes `amount` and `destinationChain` for parameters, with some native tokens for gas —> AxelarGateway handles the paying of gas and calling the correct contract and calls the `execute` function on the `DestinationBridge` contract —> from there a `txnHash` is assigned to the transaction and the transaction is minted if a certain number of approvers have approved it  —> transaction is minted and the user receives his tokens on the Destination Chain.

#### Main roles
Except for the users, there are a few governance roles that are responsible for different parts of the protocol:
DEFAULT_ADMIN_ROLE, USDY_MANAGER_ROLE, PAUSER_ROLE, MINTER_ROLE, BURNER_ROLE, LIST_CONFIGURER_ROLE

## 2. Approach taken
- Started with reading the documentation
- Read the contracts, then the documentation again, noting everything that looks suspicious
- Deep dive into the contracts

## Suggestions for improvement
For important suggestions for improvement, I'd recommend implementing a real oracle for determining the price of the USDY. Using a custom oracle like the one you implemented not only introduces a lot of Centralization risks as the owner can set the price to whatever price he wants, but also introduces external attack vectors as one of the findings I submitted. 

### Time spent:
15 hours