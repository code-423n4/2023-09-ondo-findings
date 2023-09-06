# Solidity Smart Contract Audit: rUSDY Token

## Overview

The code provided is a Solidity smart contract for a token referred to as "rUSDY," seemingly designed to be a rebasing token based on some underlying assets, most likely USDY. This contract incorporates multiple facets including roles, pausing functionalities, blocklists, allowances, and more, utilizing the OpenZeppelin upgradeable contracts library for the standard ERC20 functionalities and other utilities.

## Table of Contents

1. [Import Statements](#import-statements)
2. [Contract Declaration and State Variables](#contract-declaration-and-state-variables)
3. [Constructor and Initializer](#constructor-and-initializer)
4. [Event Declarations](#event-declarations)
5. [ERC20 Functions](#erc20-functions)
6. [Additional Public Functions](#additional-public-functions)
7. [Internal Functions](#internal-functions)
8. [Modifiers and Utility Methods](#modifiers-and-utility-methods)
9. [Points of Consideration](#points-of-consideration)

---

### Import Statements

The import statements import various upgradeable OpenZeppelin contracts for basic token functionality (`IERC20Upgradeable`, `IERC20MetadataUpgradeable`), initialization (`Initializable`), context (`ContextUpgradeable`), pause capability (`PausableUpgradeable`), and access control (`AccessControlEnumerableUpgradeable`). It also imports some custom contracts like `BlocklistClientUpgradeable`, `AllowlistClientUpgradeable`, `SanctionsListClientUpgradeable`, `IUSDY`, and `IRWADynamicOracle`.

---

### Contract Declaration and State Variables

#### State Variables

1. `mapping(address => uint256) private shares;`: Records the shares of addresses in uint256 format.
2. `mapping(address => mapping(address => uint256)) private allowances;`: Double-mapping for ERC20 allowances.
3. `uint256 private totalShares;`: Keeps track of the total shares.
4. `IRWADynamicOracle public oracle;`: The oracle interface to fetch dynamic prices.
5. `IUSDY public usdy;`: Interface for the underlying USDY token.
6. Role-specific constants: Constants for various access control roles are defined (e.g., `MINTER_ROLE`).

---

### Constructor and Initializer

The contract uses an initializer pattern which is common for upgradeable contracts. This means it has an `initializer` function and a `constructor` which calls `_disableInitializers()`.

#### `initialize()`

This is the primary function to initialize the contract. It takes various addresses like `blocklist`, `allowlist`, `sanctionsList`, `_usdy`, `guardian`, `_oracle` as arguments and initializes the contract. It internally calls `__rUSDY_init` and further `__rUSDY_init_unchained` for setting state variables and granting roles.

---

### Event Declarations

Multiple custom events like `TransferShares` and `SharesBurnt` are defined to provide richer logging capabilities.

---

### ERC20 Functions

The ERC20 functions (`totalSupply`, `balanceOf`, `transfer`, `allowance`, `approve`, `transferFrom`, etc.) have been overridden. However, they are adapted to consider the rebasing mechanism by translating amounts to/from "shares".

---

### Additional Public Functions

1. `wrap(uint256 _USDYAmount)`: Wraps USDY into rUSDY.
2. `unwrap(uint256 _rUSDYAmount)`: Unwraps rUSDY back into USDY.
3. `getSharesByRUSDY(uint256 _rUSDYAmount)`: Gets the shares by rUSDY amount.
4. `getRUSDYByShares(uint256 _shares)`: Gets the rUSDY by shares.
5. `transferShares(address _recipient, uint256 _sharesAmount)`: Transfers shares directly.
6. Role management is also implemented, most likely inherited from the parent contracts.

---

### Internal Functions

1. `_transfer`: Overloaded transfer method that adapts to the share-based balance mechanism.
2. `_approve`: Approval mechanism with additional checks.
3. `_sharesOf`: Gets the shares of a given address.
4. `_transferShares`: Internal logic for transferring shares.
5. `_mintShares`: Internal logic for minting shares.
6. `_burnShares`: Internal logic for burning shares.

---

### Modifiers and Utility Methods

1. `whenNotPaused`: Used for checking whether the contract is paused. This is inherited from `PausableUpgradeable`.

---

### Admin Setter Functions

1. `setBlockList`, `setAllowList`, and `setBlockList`: Accessed by `LIST_CONFIGURER_ROLE` which calls the internal functions to add a user to these lists. However, there is no function to remove these users. This leaves the protocol open to user error.

### Points of Consideration

1. **No Bounds Checking for Oracle**: There doesn't appear to be a mechanism to ensure the oracle's reliability or any bounds checks for the values it returns.
2. **Centralization Risks**: The contract has various roles that could execute critical functionalities. This concentration of power could pose risks.
3. **No Slippage Control**: The contract relies on an oracle for the price. Without slippage control, large transactions can induce price impacts.
4. **Error Handling**: The contract uses Solidity's native custom errors (`error UnwrapTooSmall();`). This is a more gas-efficient way to handle errors but requires the client to be capable of decoding them.
5. **Complex Multi-inheritance**: With so many parent contracts, the linearization of the contract could become problematic.
6. **Upgradability**: The contract uses OpenZeppelin's upgradeable contracts. However, there is no logic to control or restrict upgrades.
7. **Event Emittance**: It's good to see the emittance of events, providing on-chain information that can be useful for tracking.
8. **Efficiency**: The use of shares to represent ownership enables a more gas-efficient rebasing mechanism.
9. **Additional Functionalities**: The use of blocklists, sanctions lists, and allowlists adds a layer of regulatory compliance but also complexity.
10. **Pause Functionality**: The contract has pause functionality which is important for halting activities in case of emergencies.

---

# rUSDYFactory Smart Contract Audit

## Overview

The contract in focus, `rUSDYFactory`, serves as a factory for deploying instances of a token called rUSDY. It's intended to act as an administrative utility, specifically for the guardian role, to set up new rUSDY contracts using a proxy pattern for upgradability. The contract imports from OpenZeppelin's `ProxyAdmin` and `IMulticall` interfaces and uses custom contracts `Proxy` and `rUSDY`.

## Table of Contents

1. [Import Statements](#import-statements)
2. [Contract Declaration and State Variables](#contract-declaration-and-state-variables)
3. [Constructor](#constructor)
4. [Public Functions](#public-functions)
5. [Internal Functions](#internal-functions)
6. [Modifiers and Utility Methods](#modifiers-and-utility-methods)
7. [Events](#events)
8. [Points of Consideration](#points-of-consideration)

---

### Import Statements

The contract imports OpenZeppelin's `ProxyAdmin` and a custom interface `IMulticall`. It also imports custom contracts `Proxy` and `rUSDY`, which are presumably part of the same project.

---

### Contract Declaration and State Variables

#### State Variables

1. `DEFAULT_ADMIN_ROLE`: A constant defining the default admin role.
2. `guardian`: Immutable address meant to be the administrator (likely).
3. `rUSDYImplementation`: Address of the rUSDY implementation contract.
4. `rUSDYProxyAdmin`: Address of the proxy admin contract.
5. `rUSDYProxy`: Address of the token proxy contract.

---

### Constructor

The constructor takes an address `_guardian` as its argument, which is stored immutably in the `guardian` state variable.

---

### Public Functions

#### `deployrUSDY`

This function deploys a new instance of rUSDY using a proxy pattern. It sets the `rUSDYImplementation`, `rUSDYProxyAdmin`, and `rUSDYProxy` state variables and initializes the newly deployed rUSDY instance. However, this should only be called once to avoid having multiple rUSDY smart contracts, and it is missing that functionality.

#### `multiexcall`

Implements a multi-execution call, where multiple external function calls can be batch executed in a single transaction. It iterates through an array of `ExCallData` and performs calls to the target addresses with the respective data and value.

---

### Internal Functions

There are no internal functions in this contract.

---

### Modifiers and Utility Methods

#### `onlyGuardian`

This is a custom modifier that restricts the access of certain functions to the `guardian` address only.

---

### Events

#### `rUSDYDeployed`

Emitted when a new rUSDY contract is deployed. Logs the addresses of the proxy, proxy admin, and implementation contracts, along with the name and ticker symbol of the token.

---

### Points of Consideration

1. **Proxy Pattern**: The factory employs a proxy pattern to deploy instances of rUSDY. This has implications for upgradeability and potentially the encapsulation of logic.
  
2. **Centralization Risks**: Only the `guardian` has the permission to deploy new rUSDY contracts, resulting in a single point of failure.

3. **Batch Execution**: The `multiexcall` function enables batch execution of calls, which although convenient, should be carefully managed to avoid reentrancy or other complex attack vectors.

4. **Error Handling**: The contract uses Solidity's `require` statement for error handling, which is straightforward but less gas-efficient compared to using custom errors.

5. **No Event Emittance for Failure**: Currently, there is no event emitted for a failed multi-execution call in `multiexcall`.

6. **State Variable Immutability**: The `guardian` address is immutable, which means it can't be changed after contract deployment.

7. **Transfer of Ownership**: Ownership of `rUSDYProxyAdmin` is transferred to `guardian`, but the contract does not provide a mechanism to change the `guardian`.

8. **Use of Assert**: The contract uses the `assert` statement for validating the ownership transfer, which is generally discouraged as it consumes all the remaining gas if it fails.

9. **Compliance with IMulticall**: The contract claims to implement `IMulticall`, which should be verified to ensure full interface compliance.

10. **Lack of Input Validation**: There is minimal validation on the inputs to the `deployrUSDY` and `multiexcall` functions, posing potential risks.

----

# RWADynamicOracle Smart Contract Audit

## Overview

The contract `RWADynamicOracle` is engineered to manage a dynamic oracle for real-world assets (RWA), particularly focusing on setting and retrieving interest rate values within a blockchain ecosystem. The contract is authored in Solidity 0.8.16 and utilizes OpenZeppelin libraries for access control and pausability.

## Table of Contents

1. [Inheritance Structure](#inheritance-structure)
2. [Constant and State Variables](#constant-and-state-variables)
3. [Public Functions](#public-functions)
4. [Internal Functions](#internal-functions)
5. [Admin Functions](#admin-functions)
6. [Structs, Events, and Errors](#structs-events-and-errors)
7. [Interest Calculation Helper Functions](#interest-calculation-helper-functions)
8. [Potential Areas of Concern](#potential-areas-of-concern)

---

### Inheritance Structure

The contract inherits from the following:

- `IRWAOracle`: Interface defining how to get price data.
- `AccessControlEnumerable`: Manages access control.
- `Pausable`: Adds a pausability feature to the contract.

---

### Constant and State Variables

#### State Variables

1. `DAY`: A constant that defines the number of seconds in a day.
2. `ranges`: An array of `Range` structs that capture interest rate periods.
3. `SETTER_ROLE`: Byte identifier for the setter role.
4. `PAUSER_ROLE`: Byte identifier for the pauser role.

---

### Public Functions

#### `getPriceData`

An external view function that retrieves the current price and timestamp.

#### `getPrice`

This function returns the current interest rate by iterating over the `ranges`.

#### `simulateRange`

An external view function that returns the projected interest rate at a given timestamp with respect to a new range.

---

### Internal Functions

#### `derivePrice`

Calculates the price based on the daily interest rate and elapsed days.

#### `roundUpTo8`

Rounds up a given number to 8 decimal places.

---

### Admin Functions

Functions such as `setRange`, `overrideRange`, `pauseOracle`, and `unpauseOracle` are restricted to users with specific roles for administrative tasks.

---

### Structs, Events, and Errors

Structs:

1. `Range`: Captures interest rate periods and previous closing prices.

Events:

1. `RangeSet`
2. `RangeOverridden`

Errors:

1. `InvalidPrice`
2. `InvalidRange`
3. `PriceNotSet`

---

### Interest Calculation Helper Functions

Various helper functions like `_rpow`, `_rmul`, and `_mul` assist in calculations related to interest rate determination.

---

### Potential Areas of Concern

1. A thorough understanding of access control roles (`DEFAULT_ADMIN_ROLE`, `SETTER_ROLE`, `PAUSER_ROLE`) is essential.
2. The `roundUpTo8` function could introduce rounding errors.
3. Functions such as `_rpow` can be prone to underflow or overflow issues.
4. Heavy reliance on `block.timestamp`, potentially manipulable by miners.
5. Lack of error messages for custom errors hinders debuggability.
6. Use of inline assembly in `_rpow` requires careful handling.
7. The contract does not offer mechanisms for querying historical data, affecting its auditability.

---

# SourceBridge Solidity Contract Analysis

## Overview

The `SourceBridge` contract serves as an intermediary between the Axelar network and Ethereum. It facilitates the burning of tokens on Ethereum and initiates a corresponding action on a destination chain through Axelar's gateway.

## Contract Inheritance

The contract inherits from `Ownable`, `Pausable`, and `IMulticall`:

- `Ownable`: Provides basic authorization control functions, simplifying the implementation of user permissions.
- `Pausable`: Allows the contract to be paused, which stops the execution of certain functions.
- `IMulticall`: Appears to be a custom interface for batched calls, likely aiming to save on gas costs or enable complex transactions.

## State Variables

1. **destChainToContractAddr**: A mapping between destination chains and their respective contract addresses.
2. **TOKEN**: Immutable reference to an `IRWALike` token contract.
3. **AXELAR_GATEWAY**: Immutable reference to `IAxelarGateway`.
4. **GAS_RECEIVER**: Immutable reference to `IAxelarGasService`.
5. **VERSION**: A bytes32 constant indicating the contract's version.
6. **nonce**: A counter used likely for ensuring uniqueness of transactions.

## Points of Consideration

### Security

1. **Ownership Risks**: The `onlyOwner` modifier is applied to critical administrative functions, adding centralization risks.
2. **Reentrancy**: While no obvious reentrancy vulnerabilities exist, they can still potentially be introduced through `_payGasAndCallContract`.
3. **Gas Fee Checks**: The contract expects payment for gas fees but only checks for this with a `revert` statement, possibly affecting user experience.
4. **Error Handling**: Errors are defined but are not richly descriptive. It could be enhanced to offer more informative insights.

### Design Patterns

1. **Pausable Contract**: Functions are guarded by the `whenNotPaused` modifier, which is good for emergency stops.
2. **Immutable State Variables**: Utilizes immutable variables for contract references, optimizing gas costs for read operations.
3. **Nonce Management**: Utilizes a nonce for unique transaction identification, but its role and security implications are not entirely clear.

### Best Practices

1. **Admin Functionality**: Admin functions for pausing/unpausing and setting destination contract addresses are present but could be extended with a timelock or multi-signature functionality for better security.
2. **Event Logging**: Events are emitted for administrative actions, aiding in transparency and easier backtracking of operations.

### Potential Improvements

1. **Gas Fee Handling**: Consider allowing the Axelar network to refund or sponsor gas fees directly, thus eliminating the need for `msg.value` checks.
2. **Input Validation**: No obvious input validation is present for `burnAndCallAxelar`. Proper checks on the `amount` and `destinationChain` should be implemented.
3. **Multi-Signature**: Introducing multi-signature requirements for sensitive admin functions could distribute trust and reduce the risks of malicious activities.

---
i# SourceBridge Smart Contract Audit

## Overview

The contract `SourceBridge` is designed to facilitate cross-chain token transfers via the Axelar network. The contract relies on multiple interfaces and contracts, including some from external sources such as OpenZeppelin.

## Table of Contents

1. [Import Statements](#import-statements)
2. [Contract Declaration and State Variables](#contract-declaration-and-state-variables)
3. [Constructor](#constructor)
4. [Public Functions](#public-functions)
5. [Internal Functions](#internal-functions)
6. [Admin Functions](#admin-functions)
7. [Events and Errors](#events-and-errors)
8. [Points of Consideration](#points-of-consideration)

---

### Import Statements

The contract imports various other interfaces and contracts, including but not limited to:

- `IAxelarGateway`
- `IAxelarGasService`
- `IMulticall`
- `IRWALike`
- `Ownable`
- `Pausable`

---

### Contract Declaration and State Variables

#### State Variables

1. `destChainToContractAddr`: A mapping from destination chains to their respective contract addresses.
2. `TOKEN`: An immutable variable of the `IRWALike` interface.
3. `AXELAR_GATEWAY`: An immutable variable of the `IAxelarGateway` interface.
4. `GAS_RECEIVER`: An immutable variable of the `IAxelarGasService` interface.
5. `VERSION`: A constant for versioning.
6. `nonce`: A counter to ensure uniqueness of transactions.

---

### Constructor

The constructor initializes several state variables such as `TOKEN`, `AXELAR_GATEWAY`, and `GAS_RECEIVER`, thereby setting up the contract for its intended functions.

---

### Public Functions

#### `burnAndCallAxelar`

This function burns tokens from the sender and initiates a call to Axelar's gateway, transferring the necessary data to a specified destination chain.

---

### Internal Functions

#### `_payGasAndCallContract`

This function handles the payment of gas fees and initiates the actual call to the Axelar Gateway.

---

### Admin Functions

Functions such as `setDestinationChainContractAddress`, `pause`, and `unpause` are reserved for the contract owner and facilitate administrative control over the contract's behavior.

---

### Events and Errors

Several events and errors are declared to manage state changes and error handling:

1. `DestinationChainContractAddressSet`
2. `DestinationNotSupported`
3. `GasFeeTooLow`

---

### Points of Consideration

1. The contract is closely integrated with several other contracts and interfaces, necessitating an understanding of these external interactions for a comprehensive audit.
2. Utilizes OpenZeppelin's `Ownable` and `Pausable` contracts, which are generally well-audited but add layers of dependency.
3. Given the central role this contract plays in facilitating cross-chain transactions, attention must be paid to potential reentrancy attacks, proper gas fee handling, and secure nonce management.

---
# DestinationBridge Smart Contract Audit

## Overview

The contract `DestinationBridge` is designed to serve as a bridge for cross-chain transfers. The contract is dependent on a variety of other contracts and libraries from both internal and external sources like OpenZeppelin.

## Table of Contents

1. [Import Statements](#import-statements)
2. [Contract Declaration and State Variables](#contract-declaration-and-state-variables)
3. [Constructor](#constructor)
4. [Public Functions](#public-functions)
5. [Internal Functions](#internal-functions)
6. [Modifiers and Utility Methods](#modifiers-and-utility-methods)
7. [Events](#events)
8. [Points of Consideration](#points-of-consideration)

---

### Import Statements

The contract imports several other interfaces and contracts, some of which are from OpenZeppelin. Among them are:

- `IAxelarGateway`
- `IAxelarGasService`
- `AxelarExecutable`
- `IRWALike`
- `IAllowlist`
- `Ownable`
- `Pausable`
- `MintRateLimiter`

---

### Contract Declaration and State Variables

#### State Variables

1. `TOKEN`: An immutable variable of the `IRWALike` interface.
2. `AXELAR_GATEWAY`: An immutable variable of the `IAxelarGateway` interface.
3. `ALLOWLIST`: An immutable variable of the `IAllowlist` interface.
4. `VERSION`: A constant for versioning.
5. Several mappings to handle state.

---

### Constructor

The constructor initializes several state variables, setting up the contract for future use.

---

### Public Functions

#### `_mintIfThresholdMet`

This function checks if a threshold is met and then performs minting actions. 

#### `getNumApproved`

This function returns the number of approvers for a specific transaction hash.

---

### Internal Functions

#### `_attachThreshold`

This function attaches a threshold setting based on the amount for a given transaction.

#### `_approve`

This function approves a transaction based on its hash.

#### `_checkThresholdMet`

Checks if the number of approvals meets or exceeds the required threshold for a transaction.

---

### Modifiers and Utility Methods

The contract uses `onlyOwner` and `whenNotPaused` modifiers from the OpenZeppelin contracts to restrict access and functional behavior based on the contract state.

---

### Events

Several events are declared to emit state changes:

1. `ApproverRemoved`
2. `ApproverAdded`
3. `ChainIdSupported`
4. `ThresholdSet`
5. `BridgeCompleted`
6. `MessageReceived`

---

### Points of Consideration

1. The contract extensively uses state variables and requires careful consideration of state management.

2. The contract's coupling with other contracts and interfaces increases the complexity, making it critical to understand interactions.

3. It uses OpenZeppelin's `Ownable` and `Pausable`, which are well-audited, but also custom contracts that need separate auditing.

4. **Gas Costs**: High computational operations in some functions could result in elevated gas costs for users. Profiling of these costs and their impact on usability should be evaluated.

5. **Upgradability**: The contract doesn't appear to implement any upgradability patterns. This is a double-edged sword; on one hand, it increases trust in the contract, but on the other, it could become a liability if critical bugs are found post-deployment.

6. **Re-Entrancy Attacks**: While the contract doesn't directly handle Ether and primarily deals with token transfers, there could be a scope for re-entrancy attacks, particularly if one of the integrated contracts has such a vulnerability. Utilizing the re-entrancy guard pattern could mitigate this risk.

7. **Front-Running**: Functions that involve transaction ordering or price calculations could be susceptible to front-running attacks. Employing mechanisms like commit-reveal could reduce such risks.

8. **External Contract Dependence**: The contract is significantly dependent on external contracts and libraries. Any vulnerabilities or changes in these dependencies could compromise the integrity of the `DestinationBridge` contract.

9. **Ownership Centralization Risks**: The `onlyOwner` modifier is used in critical functions, which centralizes certain powers. Although OpenZeppelin's `Ownable` contract is robust, the governance model's centralization should be carefully evaluated.

10. **Input Validation**: The contract has several points where external input is accepted, which increases the risk of potential attacks like integer overflow or underflow. Proper input validation mechanisms should be employed to mitigate these.

11. **Event Logging**: While events are well-defined for a number of state-changing operations, additional events could provide more granularity and easier off-chain tracking for complex operations.

12. **Batch Operations**: The contract lacks functions that handle batch operations, potentially leading to inefficiencies in scenarios that require multiple transactions to be processed simultaneously.

13. **Error Messages**: The contract could benefit from more descriptive error messages, which could facilitate easier debugging and improve the developer experience.

14. **Time-Locking Mechanisms**: Consider implementing a time-lock for sensitive ownership and parameter-changing operations to add an additional layer of security.

15. **Rate-Limiting**: While the `MintRateLimiter` is imported, it's not clear if rate-limiting is effectively used in the contract. If it isn't, implementing rate-limiting could mitigate abuse of the system.

16. **Fallback Functions**: The contract doesn't have a designated fallback or receive function for directly interacting with Ether, which is generally a good security practice but should be noted for full understanding of the contract's capabilities and limitations.

17. **Data Availability**: The contract does not appear to offer functions for easier on-chain data availability (e.g., historical transaction hashes, past approvals, etc.), which could be useful for third-party integrations or for auditing purposes.

18. **Privacy Concerns**: The contract's public functions and events may leak sensitive business or user information. Privacy measures such as zk-SNARKs could be considered to enhance data confidentiality.

19. **Decimal Handling**: As the contract deals with token transactions, improper handling of decimals could lead to significant errors. Ensure that the contract handles decimal points consistently across all functions.

20. **Permission Layering**: The contract employs a simple permission model based on ownership. Advanced use-cases may require a more layered permissioning system, incorporating roles and responsibilities beyond just the owner.

---


### Time spent:
20 hours