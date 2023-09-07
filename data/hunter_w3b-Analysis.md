# [A-01] Any comments for the judge to contextualize your findings

My primary objective is to enhance the gas efficiency and cost-effectiveness of the protocol. Throughout my analysis, I have diligently pursued opportunities to optimize gas and have prepared a comprehensive report comprising `26` detailed gas optimization with forge test and refactoring recommendations tailored to the specific needs of the protocol.

The optimizations I have identified aim to improve gas efficiency, reduce storage reads, and overall make the smart contracts more

- Suggests using immutable for constant values like keccak256 hashes.
- Explains the gas cost difference between pre-increment (++i) and post-increment (i++) operations.
- Recommends declaring variables outside of loops to save gas when they don't need reinitialization.
- Advises sorting require() statements by gas cost for better code organization.
- Recommends using function arguments instead of multiple accesses to state variables to avoid unnecessary SLOAD operations.
- Suggests pre-calculating state variables computed with keccak256() and assigning the result to constants.
- Recommends removing unused internal functions to reduce deployment gas costs.
- Suggests packing multiple structs into fewer storage slots to reduce storage costs.
- Advises refactoring duplicated require() or if() checks into modifiers or functions for better code organization.
- Recommends using local variable caches for multiple accesses to mappings or arrays.
- Points out that abi.encodePacked() is more gas-efficient than abi.encode().
- Advises avoiding unnecessary calculations for constants to improve code efficiency.
- Recommends refactoring internal functions to avoid unnecessary SLOAD operations and improve gas efficiency.
- Suggests using do while loops for better gas optimization.
- Advises using assembly for efficient back-to-back function calls.
- Recommends adding zero address checks in constructors for proper contract initialization.
- Suggests using ternary operations instead of if-else statements for gas optimization.
- Recommends sorting Solidity operations using short-circuit mode for better code organization.
- Advises against unnecessary type casting when variables are already of the same type.
- Recommends using assembly for math operations like addition, subtraction, multiplication, and division for gas efficiency.
- Advises using hardcoded addresses instead of address(this) for clarity and optimization.
- Suggests using mappings instead of arrays when appropriate to optimize gas consumption.
- Recommends shortening arrays instead of copying to new ones when removing elements to save gas.
- Suggests using version 4.9.0 of OpenZeppelin contracts.
- Recommends using uint256(1) and uint256(2) instead of true and false boolean states for gas optimization.

# [A-02] Approach taken in evaluating the codebase

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contracts Overview**: These contracts appear to play essential roles in the protocol, such as handling bridging, managing the rebasing USDY token, deploying contracts, and providing price oracles. They also utilize various OpenZeppelin libraries for security and functionality.

- **SourceBridge.sol**

  Purpose: This contract serves as the source chain bridge for USDY.

  Libraries Used: OpenZeppelin Pausable & Ownable, Axelar Address Utils

- **DestinationBridge.sol**

  Purpose: This contract serves as the destination chain bridge for USDY.

  Libraries Used: OpenZeppelin Pausable & Ownable, Axelar Executable and String Utils

- **rUSDY.sol**

  Purpose: The rebasing USDY token contract.

  Libraries Used: OpenZeppelin Upgradeable Contracts

- **rUSDYFactory.sol**

  Purpose: The rUSDY deployment contract.

  Libraries Used: OpenZeppelin Proxy

- **RWADynamicOracle.sol**

  Purpose: This contract is the reference oracle used by rUSDY to get the price of USDY.

  Libraries Used: OpenZeppelin Access Control & Pausable

- **IRWADynamicOracle.sol**

  Purpose: Interfaces for IRWADynamicOracle.

  Libraries Used: Not applicable (N/A)

2.  **Documentation Review**:

- **Introduction to rUSDY**: rUSDY is an interest-bearing stablecoin, similar to a rebasing variant of USDY (Ondo U.S. Dollar Yield).Holders of USDY can wrap their tokens to receive rUSDY tokens proportional to the USD value wrapped.Each rUSDY token is worth $1, and its value rebases with changes in the value of USDY.The price of USDY is determined using the RWADynamicOracle.sol oracle contract.

- **Background on USDY**:USDY is an upgradeable token with transfer restrictions.Users must be on the allowlist, not on the blocklist, and not on a sanctions list to hold, send, and receive USDY. USDY represents tokenized bank deposits and appreciates in price over time due to accruing interest.

- **rUSDY**: rUSDY is a rebasing variant of USDY. Users can acquire rUSDY tokens by calling the wrap(uint256) function.
  The price of a single rUSDY token remains fixed at $1, with yield accruing in the form of additional rUSDY tokens.
  To convert rUSDY to USDY, users can call the unwrap(uint256) function. The USD value of the locked USDY in the contract is determined using the RWADynamicRateOracle.sol.

- **RWADynamicRateOracle**: This contract posts the price evolution for USDY on-chain.It accepts a range as input and calculates the current price based on the daily interest rate and days elapsed.The resulting plot of prices over time should resemble the provided FIG-01.It can return the maximum price of the previous range for all timestamps after the range has elapsed.

- **SourceBridge**: This contract is deployed on the source chain and handles calls into the Axelar Gateway for bridging USDY or an RWA token.It burns the bridging token and forwards gas along with a payload to the Axelar gas service and Axelar gateway.

- **DestinationBridge**: Deployed on the destination chain, this contract handles calls from the Axelar Gateway.Requires registration of the originating address with the Receiver contract.Received messages are queued and processed once they receive the required number of approvals.Implements a rate limit on the amount of tokens the Receiver contract can mint over a fixed duration.

3. **Graphical Analysis**: Solidity-metrics is a popular tool used to analyze and visualize Solidity code. It provides various metrics and visualizations to gain insights into the codebase's complexity and maintainability.
   [solidity-metrics](https://github.com/ConsenSys/solidity-metrics)

4. **Utilizing Static Analysis Tools**: In this phase of the analysis, Slither has already been executed as part of the static analysis process.

5. **Test Coverage Evaluation**: During this phase, I play with the tests, initially encountering challenges when attempting to run them first. However, after resolving the issues, I found the well-written tests to be quite interesting.

6. **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode, Despite the complexity of this codebase, I didn't come across any noteworthy insights during the first two modes. Consequently, I shifted my focus to the Blackboxing mode.

   - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

   - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

   - **Blackboxing Mode**: Particularly useful for complex codebases. Grasp the protocol's expected behavior through tests, and experiment with them to better understand and navigate the intricacies of the code.

# [A-03] Architecture recommendations

The following architecture improvements and feedback could be considered:

- **RWADynamcRateOracle contract**

  1. **Clarification on Range**: Provide a clear explanation of what constitutes a "Range" in the context of this contract. It would be helpful to define the parameters that make up a Range (e.g., daily interest rate, elapsed days, last set price) and how these parameters are determined.
  2. **Use of Time**: Provide information on how time is tracked within the contract. Clarify whether it relies on block timestamps or external time oracles, as this can affect the accuracy of price evolution.

  3. **Units and Scaling**: Specify the units and scaling used in the formula. For instance, clarify whether daily interest rates are expressed as decimals or percentages. This ensures that readers can apply the formula correctly.

- **Transfer Restrictions**: Apply the same transfer restrictions to rUSDY as those applied to USDY. This consistency ensures that both tokens comply with regulatory requirements and security measures.

- **DestinationBridge contract**: Implement rate limiting mechanisms and thresholds in the DestinationBridge contract to control the minting of tokens. This prevents potential abuse and ensures responsible token minting.

  1. **Dynamic Approval Thresholds**: Implement a flexible approval mechanism that adapts to different scenarios. Consider allowing dynamic approval thresholds based on factors such as the source chain, the amount being bridged, or user-defined parameters.

  2. **Rate Limiting**: The rate limit implementation should be configurable and adjustable by administrators. Consider using parameters that allow for changing the rate limit over time to accommodate changing needs and network conditions.

- **USDY contract**

  1. **Yield Variability**: Acknowledge that the yield on bank deposits can vary over time due to changing market conditions. Implement a mechanism to update the interest rate dynamically to reflect the current yield accurately.

  2. **Interest Accrual**: Ensure that the USDY contract accurately accrues interest over time, reflecting the yield earned by tokenized bank deposits. Implement a clear and transparent method for interest calculation.

- **Transaction Queue**: Describe the queueing mechanism for incoming bridging messages. Explain how messages are queued and what criteria are used to determine their processing order.

- **Approvals and Thresholds**: Explain the concept of approvals required for processing messages and how these thresholds can vary based on factors such as source chain and token amount.

# [A-04] Codebase analysis

1. **SourceBridge.sol**: This contract serves as a bridge between different networks, facilitating the transfer of tokens from one chain to another through the use of the AxelarGateway and AxelarGasService contracts. It enforces certain conditions and provides administrative functions for managing the bridge's configuration. The ownership structure ensures that only authorized parties can perform administrative actions.

   - **Contract Inheritance**: SourceBridge inherits from two contracts: Ownable and Pausable. This means it incorporates the functionality of these contracts. Ownable provides ownership management, and Pausable allows the contract to be paused and unpaused.

   - **State Variables**: The contract defines several state variables, including:
     destChainToContractAddr: A mapping that associates destination chain names with contract addresses in string format.
     TOKEN: An immutable reference to a token contract (IRWALike).
     AXELAR_GATEWAY: An immutable reference to the AxelarGateway contract.
     GAS_RECEIVER: An immutable reference to the AxelarGasService contract.
     VERSION: A constant bytes32 variable representing the contract version.
     nonce: A monotonically increasing nonce used for transaction tracking.
   - **Constructor**: The constructor initializes the contract with addresses of the token contract, AxelarGateway, AxelarGasService, and an owner address.

   - **burnAndCallAxelar Function**: This function is used to burn tokens on the source chain and then call the AxelarGateway contract to mint tokens on a destination chain.
     It takes amount and destinationChain as parameters.
     It checks whether the provided destination chain is supported and if the sent gas fee is sufficient.
     It burns the specified amount of tokens from the sender's address.
     It encodes a payload and invokes the \_payGasAndCallContract private function.

   - **\_payGasAndCallContract Function**:A private helper function used to pay gas fees and call the AxelarGateway contract on the destination chain.
     It takes destinationChain, destContract, and payload as parameters.
     It sends the gas fee to the GAS_RECEIVER contract and then calls the AXELAR_GATEWAY contract with the specified parameters.

   - **Admin Functions**: These functions are meant for administrative purposes and can only be called by the contract owner.
     setDestinationChainContractAddress sets the mapping of destination chains to contract addresses.
     pause and unpause functions allow the owner to pause and unpause the contract, which can be useful for maintenance or in emergency situations.
     multiexcall allows the owner to make arbitrary batched calls to external contracts.

   - **Events and Errors**: The contract defines an event DestinationChainContractAddressSet for logging when the destination chain to contract address mapping is set.
     It also defines custom errors (DestinationNotSupported and GasFeeTooLow) that can be raised in certain conditions to revert transactions with specific error messages.

2. **DestinationBridge.sol**: This contract serves as a destination bridge for tokens, allowing them to be transferred from a source chain to a destination chain through the Axelar Gateway. It enforces various conditions, approvals, and thresholds for minting tokens on the destination chain. The ownership structure ensures that only authorized parties can perform administrative actions, and the pausing mechanism allows for emergency control over the contract's functionality.

   - **Contract Inheritance**:DestinationBridge inherits from several contracts: AxelarExecutable, MintTimeBasedRateLimiter, Ownable, and Pausable. This means it incorporates the functionality of these contracts.
     AxelarExecutable and MintTimeBasedRateLimiter likely provide functionality related to execution by an Axelar Gateway and rate limiting, respectively.
     Ownable provides ownership management, and Pausable allows the contract to be paused and unpaused.

   - **State Variables**:The contract defines several state variables, including:
     TOKEN: An immutable reference to a token contract (IRWALike).
     AXELAR_GATEWAY: An immutable reference to the AxelarGateway contract.
     ALLOWLIST: An immutable reference to an allowlist contract.
     approvers: A mapping of addresses to booleans to track approved approvers.
     chainToApprovedSender: A mapping of destination chains to approved sender hashes.
     isSpentNonce: A mapping to track spent nonces.
     VERSION: A constant bytes32 variable representing the contract version.
     Various mappings and structs related to tracking transactions and thresholds.

   - **Constructor**:The constructor initializes the contract with various parameters, including addresses of the token contract, AxelarGateway, allowlist, owner, mint limit, and mint duration.
     It also initializes some state variables.

   - **Axelar Functions**:\_execute: An internal function that is executed when the contract is called by the Axelar Gateway. It validates the payload and processes the transaction accordingly.

   - **Internal Functions**:\_attachThreshold: Attaches a specific threshold to a given txnHash.
     \_approve: Approves and conditionally mints for a given transaction. Approval is conditional on the approver not having previously approved the transaction.
     \_checkThresholdMet: Checks if the approval threshold has been met for a given transaction.

   - **Protected Functions**:approve: Allows approvers to approve and conditionally mint transactions.
     addApprover and removeApprover: Admin functions to add and remove approvers.
     addChainSupport: Admin function to allow bridge calls originating from a given address on a given chain.
     setThresholds: Admin function to set transaction thresholds for a chain.
     setMintLimit and setMintLimitDuration: Admin functions to set mint limits and durations.
     pause and unpause: Admin functions to pause and unpause the contract.
     rescueTokens: Admin function to rescue ERC20 tokens sent to the contract.

   - **Public Functions**:\_mintIfThresholdMet: Internal function to mint a transaction if it has passed the threshold for the number of approvers.
     getNumApproved: External view function to get the number of approvers for a given txnHash.

   - **Structs and Events**:Several structs and events are defined in the contract for various purposes, including tracking thresholds, approvals, supported chains, and completed bridge transactions.

   - **Errors**:Custom error messages are defined in the contract to provide specific error information in case of contract failures.

3. **rUSDY.sol**: Controlling token transfers based on blocklists, allowlists, and sanctions lists. Additionally, it supports the wrapping and unwrapping of tokens based on shares and a dynamic price oracle.

   - **Inheritance**:The contract inherits from several OpenZeppelin libraries and interfaces, including Initializable, ContextUpgradeable, PausableUpgradeable, AccessControlEnumerableUpgradeable, BlocklistClientUpgradeable, AllowlistClientUpgradeable, SanctionsListClientUpgradeable, IERC20Upgradeable, and IERC20MetadataUpgradeable.

   - **State Variables**:shares: A mapping that tracks the number of shares held by each account.
     allowances: A mapping that tracks token allowances granted by token holders.
     totalShares: The total number of shares in existence.
     oracle: An address representing the Oracle used to update the price of USDY.
     usdy: An address representing the USDY token contract.
     BPS_DENOMINATOR: A constant representing the denominator used for scaling up USDY amounts to shares.

   - **Roles**:The contract defines several roles using bytes32 constants, including USDY_MANAGER_ROLE, MINTER_ROLE, PAUSER_ROLE, BURNER_ROLE, and LIST_CONFIGURER_ROLE. These roles are used for access control.

   - **Initialization**:The contract can be initialized with specific addresses for the blocklist, allowlist, sanctions list, USDY token, guardian, and Oracle.

   - **Token Transfer Functions**:transfer: Allows token holders to transfer tokens to other addresses.
     transferFrom: Allows token holders to transfer tokens on behalf of another address (following the allowance mechanism).
     approve: Allows token holders to set an allowance for another address to spend their tokens.
     increaseAllowance: Allows token holders to increase the allowance for another address.
     decreaseAllowance: Allows token holders to decrease the allowance for another address.

   - **Balance and Supply Functions**:balanceOf: Returns the token balance of a specific address.
     totalSupply: Returns the total supply of the token.

   - **Share Transfer Functions**:transferShares: Allows token holders to transfer shares to other addresses, representing a proportional ownership of USDY.
     wrap: Allows users to wrap USDY tokens into rUSDY tokens by minting new shares.
     unwrap: Allows users to unwrap rUSDY tokens into USDY tokens by burning shares.

   - **Oracle and Price-Related Functions**:setOracle: Allows the contract admin to set the Oracle address for updating the USDY price.

   - **Admin Functions**:burn: Allows admin to burn rUSDY tokens from any account.

   - **Pausing Functions**:pause: Allows admin to pause the contract, preventing most token transfers.
     unpause: Allows admin to unpause the contract.

   - **List Configuration Functions**:setBlocklist: Allows admin to set the blocklist address.
     setAllowlist: Allows admin to set the allowlist address.
     setSanctionsList: Allows admin to set the sanctions list address.

4. **rUSDYFactory.sol**: This factory for deploying upgradeable instances of the rUSDY token, along with the necessary proxy and admin contracts. It's designed to be used in a specific ecosystem, and it enforces permissions based on the guardian address.

   - **State Variables**:guardian: An immutable state variable storing the address of the guardian, set during contract deployment.
     rUSDYImplementation: A state variable that will store the address of the implementation contract for the rUSDY token.
     rUSDYProxyAdmin: A state variable that will store the address of the proxy admin contract for the rUSDY token.
     rUSDYProxy: A state variable that will store the address of the proxy contract for the rUSDY token.

   - **Constructor**: The constructor of this contract takes the guardian address as a parameter and assigns it to the guardian state variable.

   - **deployrUSDY Function**: This function is used to deploy an instance of the rUSDY token and related contracts. It takes several addresses as parameters, including a blocklist, an allowlist, a sanctions list, the address of USDY, and an oracle address. The purpose of this function is to set up the entire ecosystem around the rUSDY token. Here's a summary of what this function does:

   It deploys an rUSDY implementation contract.
   It deploys a ProxyAdmin contract.
   It deploys a TransparentUpgradeableProxy contract and links it to the rUSDY implementation and ProxyAdmin.
   It initializes the rUSDY contract with various parameters.
   It transfers ownership of the ProxyAdmin contract to the guardian.

   - **multiexcall Function**: This function allows for arbitrary batched calls to other contracts. It takes an array of ExCallData structs as a parameter, where each struct contains target contract, data, and value. This function is restricted to be called only by the guardian.

   - **rUSDYDeployed Event**: An event is emitted when an upgradable rUSDY contract is deployed. It provides the addresses of the proxy contract, proxy admin contract, and implementation contract, along with the name and ticker of the token.

   - **Modifier onlyGuardian**: This modifier restricts certain functions (like deployrUSDY and multiexcall) to be called only by the guardian address specified during contract deployment.

5. **RWADynamicOracle.sol**: This contract as an oracle for calculating the dynamic price of the USDY token based on predefined price ranges and daily interest rates. It provides role-based access control to certain functions and can be paused or unpaused as needed. The contract also includes functions for simulating price derivations and rounding calculations.

   - **Public Functions**: getPriceData(): A function that returns the current price of USDY along with the current timestamp.
     getPrice(): Returns the current price of USDY. It calculates the price based on the defined ranges and the current timestamp.
     simulateRange(...): An external helper function used to simulate the derivation of prices given a specific range and timestamp.

   - **Admin Functions**: setRange(...): Allows an admin to set a new price range for USDY. It checks that the new range's end timestamp is greater than the last range's end.
     overrideRange(...): Allows an admin to override a previously set range, updating its properties.
     pauseOracle(): Allows the pauser role to pause the oracle, preventing price calculations.
     unpauseOracle(): Allows the default admin role to unpause the oracle.

   - **Internal Functions**: derivePrice(...): An internal helper function used to derive the price of USDY for a given range and timestamp.
     roundUpTo8(...): An internal function to round the derived price to the 8th decimal place, rounding up 5 or greater.

   - **Structs and Events**: Range: A struct that represents a price range with start and end timestamps, a daily interest rate, and the previous range's close price.
     RangeSet and RangeOverriden: Events emitted when a range is set or overridden.

   - **Custom error types**: InvalidPrice, InvalidRange, and PriceNotSet.
     Interest Calculation Helper Functions: A set of functions for interest rate calculations, including \_rpow, \_rmul, and \_mul. These functions are used internally for price calculations.

# [A-05] Centralization risks

1. **SourceBridge.sol**

   - **Nonce Management**: The contract uses a nonce to ensure that each transaction is unique. If the owner can manipulate or reset the nonce, it could lead to centralization risks, including potential replay attacks.

   - **Admin-Only Functions**: Several critical functions, such as setDestinationChainContractAddress, pause, and unpause, can only be called by the contract owner. If these functions are not used responsibly, it can lead to centralization of control.

   - **Admin-Controlled External Calls**: The multiexcall function allows the contract owner to make arbitrary batched external calls. If these calls are used maliciously or irresponsibly, it can lead to centralization risks, such as draining user funds or compromising security.

2. **DestinationBridge.sol**

   - **Approver System**: The contract implements an approver system where specific addresses can approve transactions. If the list of approvers is controlled by a single entity or a small group, it can lead to centralization risks, especially if those entities are malicious or compromised.

   - **Threshold Management**: The contract allows the owner to set and manage transaction thresholds. If the owner has control over these thresholds, it can potentially manipulate the system to favor certain transactions or users, leading to centralization risks.

   - **Mint Limit and Duration**: The contract has an adjustable mint limit and duration. If these parameters are controlled by the owner without proper oversight or checks, it can result in centralization of control over the minting process.

3. **rUSDY.sol**

   - **Oracle Selection**: The contract allows the owner (USDY_MANAGER_ROLE) to set the oracle address. The owner has the authority to switch to a different oracle address, potentially centralizing control over the price oracle, which is critical for determining token values.

   - **Blocklist, Allowlist, and Sanctions List**: The owner controls the addresses that determine which other addresses are allowed or blocked. This can centralize control over who can interact with the contract.

4. **rUSDYFactory.sol**

   - **Ownership Transfer**: The contract transfers ownership of the proxy admin contract to the guardian without any checks or decentralized decision-making processes. This transfer of ownership should be carefully managed to avoid centralization.

# [A-06] Systemic risks

1. **Oracle Dependency**: The rUSDY contract relies on an external oracle contract (RWADynamicOracle.sol) to determine the current price of USDY. Systemic risks can arise if this oracle contract malfunctions, provides inaccurate data, or becomes unavailable. These risks could impact the stability of rUSDY's value and its functionality.

2. **Centralization of Price Data**: The price evolution data for USDY is stored on-chain, and a trusted admin can set this data. If the admin is controlled by a centralized entity, there is a systemic risk of price manipulation or data inaccuracies that could affect rUSDY's value.

3. **Rate Limiting**: The DestinationBridge contract implements rate limiting for minting tokens. Systemic risks can occur if the rate limit is not appropriately calibrated or if it impacts the liquidity and availability of rUSDY tokens.

4. **Cross-Chain Risks**: The contracts involve bridging tokens between source and destination chains. Systemic risks can occur if there are issues with the bridging mechanism, including misconfigurations, delays in processing, or loss of tokens during the cross-chain transfer process.

5. **Destination Chain Mapping**: The SourceBridge.sol contract allows the owner to set the mapping of destination chains to contract addresses. If this mapping is manipulated or misconfigured, it can lead to funds being sent to the wrong destination.

6. **Arbitrary Batched Calls**: The `SourceBridge.sol` contract includes a function for arbitrary batched calls (multiexcall) that allows the owner to execute external contract calls. This function poses a systemic risk if it is used maliciously or if the external calls lead to unintended consequences.

7. **Chain Support Mapping**: The DestinationBridge.sol contract allows the owner to specify which source chains are supported. If this mapping is manipulated or misconfigured, it can lead to funds being sent from unsupported chains.

8. **Blocklist and Allowlist**: The `rUSDY.sol` contract relies on blocklists and allowlists. If these lists are not updated appropriately or are manipulated, it can lead to incorrect actions being taken by the contract, affecting users' funds.

9. **Time-Related Risks**: The RWADynamicOracle.sol contract relies on timestamps for various operations. Ensure that the time-based logic is robust and cannot be manipulated or exploited.


### Time spent:
10 hours