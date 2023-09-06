## Advanced Analysis Report for [Ondo](https://github.com/code-423n4/2023-09-ondo) by K42
### Overview
- [Ondo](https://github.com/code-423n4/2023-09-ondo) Ondo is a DeFi platform that specializes in yield optimization and asset management, with a focus on cross-chain interoperability. For this audit, I focused on gas optimizations, therefore security recommendations are brief in this analysis. 

### Understanding the Ecosystem:

**Key Contracts**:
- [SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol): Handles asset transfers from the source chain.
- [DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol): Manages asset transfers on the destination chain.
- [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol): Represents the synthetic USDY asset.
- [rUSDYFactory.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol): Factory contract for creating rUSDY tokens.
- [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol): Oracle for real-world asset pricing.
- [IRWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/IRWADynamicOracle.sol): Interface for the [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol).

**Inter-Contract Communication**:

- [SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol) <-> [DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol): Asset transfer and validation.
- [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol) <-> [rUSDYFactory.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol): Token creation and management.
- [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol) <-> [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol): Price feed for synthetic assets.

### Codebase Quality Analysis:

**Security**:
- Role-based access control is implemented well.
- Use of OpenZeppelin libraries for standard functionalities like ERC20. 

### Architecture Recommendations:
- Implement a proxy pattern for contract upgradability to allow for future improvements without requiring a redeployment of the entire contract.
- Consider using a Layer 2 solution like Optimism or zk-Rollups for scalability, especially for the bridge contracts.

### Centralization Risks:
- [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol) serves as a single point of failure. If compromised, it could manipulate asset prices.
- Admin roles have the ability to pause contracts and modify key parameters, such as interest rates in [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol), without community input.

### Mechanism Review:

**Asset Transfer**: 
- Secure but dependent on external oracles.
- Needs more decentralization in oracle selection.

**Token Management**:
- Factory pattern used for token creation.
- Consider adding a token burning mechanism.

**Role-Based Access Control**:
- Implemented but could be optimized for gas.
- Consider using a more dynamic role management system.

### Systemic Risks:

**Specific Risks and Mitigations in Key Contracts**:

[SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol): 
- **Risks**: Front-running attacks during asset transfer.
- **Mitigations**: Implement a commit-reveal scheme.

[DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol):
- **Risks**: Incorrect asset mapping.
- **Mitigations**: Double-checking through off-chain services.

[RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol):
- **Risks**: Oracle manipulation.
- **Mitigations**: Multiple data sources and median calculation.

### Areas of Concern
- Oracle dependency.
- Lack of upgradability.
- Centralized role management.

### Recommendations
- Implement Chainlink oracles for more secure and decentralized price feeds.
- Implement a DAO for community governance.

### Contract Details
Here are function interaction graphs for each contract I made to better visualize interactions: 

- Link to **[Graph](https://pasteboard.co/h1eOiGX61K6I.png)** for [SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol).

- Link to **[Graph](https://pasteboard.co/BG6GhZ9VyBfv.png)** for [DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol).

- Link to **[Graph](https://pasteboard.co/TIaYUQIa6j2E.png)** for [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol).

- Link to **[Graph](https://pasteboard.co/mb3Oqpep6yGy.png)** for [rUSDYFactory.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol).

- Link to **[Graph](https://pasteboard.co/u1G484B1S7H9.png)** for [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol).

- Link to **[Graph](https://pasteboard.co/FhmT2ThcJHvQ.png)** for [IRWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/IRWADynamicOracle.sol).

### Conclusion
- While Ondo's architecture is robust in its core functionalities, it has several areas that require attention, particularly in terms of decentralization, security, and upgradability. These improvements are crucial for mitigating systemic risks and ensuring long-term sustainability. 

### Time spent:
20 hours