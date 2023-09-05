# Analysis of the TapiocaDAO Contest

# Introduction
- The Ondo Finance protocol is introducing a rebasing token for their existing yielding token USDY.
  - The purpose of the rUSDY token is to make easier the process to transact since 1 rUSDY is worth 1 dollar, in comparisson, the value of the USDY token varies overtime, which that might cause some troubles when transacting and estimating costs.
  - Any user who hold USDY can wrap their tokens for rUSDY, the rUSDY balances will rebase as USDY accrues values.
  - User can unwrap their rUSDY and receive the equivalent of their balances in USDY based on the current price of USDY at the moment of unwrapping.
  - rUSDY calls into RWADynamicRateOracle.sol in order to fetch the current price of USDY and determine its worth in USD

- The RWADynamcRateOracle contract is used to post price evolution for USDY on chain

- In addition to the new rebasing token, the Ondo Protocol also introduces an implementation to bridge assets among different chains by using the Axelar Network.
  - Basically they are taking advantage of the security and infraestructure of the Axelar Network to perform the cross chain message communication.
  - Two contracts were created for this implementation, a SourceBridge and a DestinationBridge, both of these contracts will be deployed on each chain that will be supported, the bridges are multiconnected, meaning, one SourceBridge can send crosschain messages to different DestinationBridges, or in other words, each DestinationBridge can receive messages from the different SourceBridges.
  - This solution allows the Ondo Team to offer to their users the possibility to bridge their USDY holdings among different chains.

## Analysis of the Codebase
- In general, the codebase is very hardened, I didn't find any high vulnerabilities that allows attackers to increase the supply of rUSDY, or steal other user's USDY wrapped tokens.
- Also, the bridge system is very robust, the protocol did a good work implementing their bridge solution with the Axelar Network. The Axelar Network is basically in charge of the cross chain communication layer, which allows the Ondo Protocol to bridge their USDY token among different chains.
![Bridge Implemenation Using the Axelar Network as the Crosschain Message Layer](https://res.cloudinary.com/djt3zbrr3/image/upload/v1693933912/OndoFinance/BurningAndMintingUSDY_UsingAxelarNetwork.png)


- The oracle is designed to be updated manually, and the implementation correctly returns the requested price, which is required by the rUSDY contract to determine the value in USD of the USDY token.

## Architecture Feedback, Centralization & Systemic Risks
- The entire architecture is very robust and well implemented, using the Axelar Network as the crosschain message layer removes a lot of burden to bridge assets and increases the security because to execute operation on a Destination Chain, the Axelar Network must validate that a message was sent from the SourceBridge, otherwise, tokens can't be minted in the Destination Chain, thus, if the message was sent from a valid SourceBridge it means that tokens have been already burnt.

- The existing oracle is quite centralized because the price is updated manually, but this is by design because the assets that are tracked varies and right now I believe there is not an existing oracle that tracks the value of those assets.

- There are no major problems in the rUSDY contract, the logic to wrap,unwrap and transfer rUSDY is pretty solid. Scaling up 1:10000 the ratio of USDY:rUSDY facilitates the operations in the rUSDY contract.

## Recommendations
- Based on the issues I caught on the codebase I'd recommend the protocol/devs to keep in mind that in most of the blockchains, the tx are not executed as they are sent, which opens up the doors to frontrun some tx that could end up affecting other transactions, this could lead to undesired results such as users gainning more allowance or preventing the burning of their tokens if the protocol has decided to seize their USDY tokens for any reason.

- Yes, the codebase is quite centralized, but that is by design since Ondo is an institutional-grade finance tokenizing short-term US Treasuries and bank demand deposits, I believe the relationship between the protocol and their users will be bounded by a lot of legal paper work, thus, Ondo protocol may not be incentivized to abuse the privileges they have over the code.

- A personal recommendation would be to incentivize the protocol to explore how could they make more decentralize and autonomous their codebase, take a look at existing decentralized solutions that pulls data from the real world into the blockchain (chainlink is currently the leader on this field).

# Time Spent on the Audit
- I spent around 5 days on this audit, each day I put in around 7-10 hours of work. In total that would be around 40 - 50 hours.

### Time spent:
50 hours