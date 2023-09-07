# Ondo Finance Smart Contracts Analysis

## Description overview of Ondo Finance

``Ondo Finance`` is a blockchain platform that offers investment opportunities with cryptocurrencies ``backed`` by U.S. dollar bank deposits ``(USDY)``. ``USDY`` earns interest over time, increasing its value. ``Ondo Finance`` introduces ``rUSDY``, a variant of ``USDY`` that automatically adjusts in quantity to reflect interest but maintains a nominal value of 1`` dollar`` per ``token``. ``Oracles`` track the value of ``USDY``. ``Ondo Finance`` enables asset transfers between ``blockchains`` through ``bridge`` contracts.

![Ondo](https://github.com/catellaTech/Ondo/blob/main/Ondo1.drawio.png?raw=true)

## 1- System Overview

### **Scope**

- [SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol)This ``contract`` facilitates the transfer of tokens between different blockchains using the ``Axelar protocol``, ensuring that tokens burned on one chain are minted on the corresponding destination chain, provided certain conditions are met, and the necessary gas is paid. Additionally, it provides administrative functionalities and a ``pause mechanism`` when needed.

- [DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol): This ``contract`` functions as the destination ``chain bridge`` for ``USDY`` tokens. Its primary purpose is to facilitate the seamless transfer of ``USDY`` tokens from a source blockchain to the destination blockchain where this contract resides. This ``bridge`` contract plays a pivotal role in enabling ``cross-chain`` interoperability and ensuring that ``USDY`` tokens can be securely and efficiently moved between different ``blockchain networks``.

- [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol): This ``contract`` serves as the backbone for an ``interest-bearing`` token where users can ``wrap`` and ``unwrap`` their ``USDY`` tokens to ``earn interest``, and it includes features for ``access control`` and ``address management``.

- [rUSDYFactory.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol): is This is a ``factory contract`` that allows the ``deployment`` of ``upgradable instances`` of the ``rUSDY token`` contract. It is managed by a ``guardian address``, and the deployment process involves creating an ``implementation contract``, a ``proxy admin contract``, and a ``proxy contract`` for the ``token``. The code is designed to facilitate the ``upgradeability`` of the ``rUSDY`` token and includes functions for batched ``external calls``.

- [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol): this ``contract`` is a ``dynamic Oracle`` that provides the price of ``USDY`` based on ``configured time ranges`` and ``daily interest rates``. It also ``includes mechanisms`` for ``pausing`` and ``access control`` to manage the ranges and ``contract`` operation.

- [IRWADynamicOracle](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/IRWADynamicOracle.sol): thi is an ``interface`` that sets a standard for any contract wishing to provide information about the price of ``RWA`` (Asset-Backed Real World Assets) or similar assets. ``Contracts`` that implement this ``interface`` must provide a ``getPrice()`` function that returns the current price of ``RWA``. The ``interface`` does not contain implementation logic but simply establishes a common structure for communication with ``dynamic asset price oracles``.

### **Privileged Roles**

The ``SourceBridge`` contract uses the ``onlyOwner`` modifier to restrict access to important administrative functions such as [configuring destination chain-to-contract](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L121), [pausing](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L136) / [unpausing](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L145) the contract, and [executing batch calls](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L160) to other ``contracts``.

``onlyOwner``  is also present in these functions of the contract ``DestinationBridge`` ------> [addApprover](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L210), [removeApprover](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L220), [addChainSupport](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L234), [setThresholds](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L255), [setMintLimit](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L286), [setMintLimitDuration](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L295), [pause](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L304), [unpause](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L313), and [rescueTokens](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L322).

* Only the ``address`` that ``deploys`` the ``contract`` and possesses the ``owner role`` can execute these functions. The ``owner`` has ``full control`` over the contract and its administrative functions.

In the ``rUSDY contract``, we found some ``roles``, such as:
```solidity
   /// @dev Role based access control roles
  bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
  bytes32 public constant LIST_CONFIGURER_ROLE =
    keccak256("LIST_CONFIGURER_ROLE");

```
- **USDY_MANAGER_ROLE**: This role has access to functions related to the ``management`` of the ``USDY`` token, such as ``changing the oracle`` that ``updates the USDY price``.

- **MINTER_ROLE**: This role has the ability to ``create new rUSDY tokens`` by minting them. It is responsible for the ``wrap`` function, which allows users to convert`` USDY into rUSDY``.

- **PAUSER_ROLE**: This role can ``pause`` and ``unpause`` the operations of the ``rUSDY`` contract. When the contract is paused, transfers and other operations are not available.

- **BURNER_ROLE**: This role allows administrators to ``burn`` (delete) ``rUSDY`` from a specific account using the ``burn`` function.

- **LIST_CONFIGURER_ROLE**: This role is used to configure the ``blocklist``, ``allowlist``, and ``sanctions list``. Administrators with this role can ``change`` the ``addresses`` in these ``lists``.

The ``rUSDYFactory`` contract defines the following privilege roles:

- **DEFAULT_ADMIN_ROLE**: This role represents the default administrator role. It has access to certain administrative functions in the contract.
```solidity
bytes32 public constant DEFAULT_ADMIN_ROLE = bytes32(0);
```
- **onlyGuardian**: This modifier ensures that only the ``guardian`` address has access to certain functions of the contract, such as ``deployrUSDY``.
```solidity
  modifier onlyGuardian() {
    require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");
    _;
  }
```

The ``RWADynamicOracle`` contract defines the following roles of privileges:
```solidity
  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```
- **SETTER_ROLE**: This role allows certain addresses to set a ``price range`` for ``USDY``.
- **PAUSER_ROLE**: This role allows certain addresses to ``pause`` and ``unpause`` the ``operation of the oracle``.

We asked the sponsors about who will be responsible for these roles, and this was their response:

```shell
ali2251 — 09/5/2023 at 10:22
it will be managed by multisigs controlled by the protocol
```
Given the ``level of control`` that these ``roles`` possess ``within the system``, users must place full trust in the fact that the ``entities`` overseeing these ``roles`` will always act ``correctly`` and in the ``best interest`` of the ``system`` and its ``users``.
## 2- Codebase analysis through diagrams.
We have decided to create ``diagrams`` detailing each part of the ``functions`` of the ``smart contracts`` provided by the ``Ondo Finance`` protocol and to create a summary of the functionality of each of the ``contracts``. This approach proves to be ``effective`` for ``us`` as it allows us to thoroughly understand all the functionalities of each ``contract`` and visually ``document`` what we have ``comprehended`` through ``diagrams``. Furthermore, we believe that this approach can be useful in enhancing the understanding of the ``contracts`` among ``developers``, ``auditors``, and ``users``.

- In this audit, the protocol provided `5 contracts` and 1 ``Interface``. Here's a detailed diagrams of the essential components of each ones:
### Contracts:
### 1. SourceBridge:

![SourceBridge](https://github.com/catellaTech/Ondo/blob/main/Ondo2SourceBridge.drawio.png?raw=true)
### 2. DestinationBridge:

![DestinationBridge](https://github.com/catellaTech/Ondo/blob/main/Ondo3DestinationBridge.drawio.png?raw=true)
### 3. rUSDY:

![rUSDY](https://github.com/catellaTech/Ondo/blob/main/Ondo4rUSDY.drawio.png?raw=true)
### 4. rUSDYFactory:

![rUSDYFactory](https://github.com/catellaTech/Ondo/blob/main/Ondo5rUSDYFactory.drawio.png?raw=true)

### 5. RWADynamicOracle:

![RWADynamicOracle](https://github.com/catellaTech/Ondo/blob/main/Ondo6RWADynamicOracle.drawio.png?raw=true)

## 3- Architecture Feedback 
The ``architecture`` of the ``Ondo Finance`` project seems solid in general, but there is always room for ``improvements`` and ``adjustments``. Here are some areas that could be improved:

- **Upgradeability**: Since the contracts ``rUSDY``, ``rUSDYFactory`` are upgradeable, having a clear and secure process for performing ``upgrades`` is important. It may also be useful to implement time limits for ``upgrades`` or allow a majority of users to approve ``upgrades``.

- **Governance**: Consider incorporating a ``governance system`` to allow users to vote on ``key decisions``, such as changes in ``interest rates`` or ``adjustments`` to system parameters.

- **Oracles and Data Consistency**: The consistency and accuracy of ``oracle`` data are crucial. You can explore the possibility of using ``multiple oracles`` or implementing ``consensus mechanisms`` to ensure data reliability.

- ``Testing and Simulations``: Even though the project implements several ``tests``, upon reviewing the ``codebase``, there are several crucial functions that don't, such as ``rUSDY:transferFrom``. Conduct thorough ``testing`` of ``all contracts and functions`` and ``simulations`` to understand how they will behave under various market conditions. This can help anticipate potential issues.


## 4- Documents

- The documentation of the `Ondo Finance` project is quite comprehensive and detailed, providing a solid overview of how `Ondo` is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as `diagrams`, to gain a deeper understanding of how different contracts interact and the ``functions`` they implement. With considerable ``enthusiasm``, we have dedicated some days to creating ``diagrams`` for each of the contracts.
We are confident that these ``diagrams`` will bring significant value to the protocol as they can be seamlessly integrated into the existing `documentation`, enriching it and providing a more comprehensive and detailed understanding for `users`, `developers` and `auditors`.

## 5- Systemic & Centralization Risks

Here's an analysis of potential systemic and centralization risks in the provided contracts:

- **Loss of Funds Risk**: If the contract stores or manages digital assets (such as tokens), there is an inherent risk of funds being lost due to errors in the contract's logic or malicious attacks.

- The protocol uses a `proxy`, and according to the `Documentation`, the type of `proxy` is:
    * [rUSDYFactory.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L33C1-L34C81)
  
    ```shell
  * 3) TransparentUpgradeableProxy - OZ, proxy contract. Admin is set to `address(proxyAdmin)`.
  *                                          `_logic' is set to `address(rUSDY)`.
    ```

  If the logic controlling the `proxy` is not implemented correctly, there could be `vulnerabilities` that allow an attacker to modify the underlying contract or the `proxy` itself in an unintended way.

- **Price Manipulation Risk**: Since these contracts are designed to calculate the price of ``USDY`` based on ``interest rates`` and ``price ranges``, there is a risk that ``parameters`` may be ``manipulated`` or that ``input data`` may be compromised, potentially leading to ``incorrect prices``.

- **Centralized Administration Risk**: If the contract has ``administrator roles`` that can modify ``settings`` or ``pause the contract``, there is a risk that these capabilities may be used improperly or become targets for attacks.

- **Third-Party Dependency Risk**: Contracts rely on external data sources, such as [@openzeppelin/contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/tree/v4.9.2), and there is a risk that if any `issues` are found with these `dependencies` in your contracts, the `Ondo finance` protocol could also be `affected`.

  - We observed that `old versions` of `OpenZeppelin` are used in the project, and these should be updated to the `latest version`:

  ```solidity
    29: "@openzeppelin/contracts": "^4.8.3",
    30: "@openzeppelin/hardhat-upgrades": "1.22.1",
  ```
  The latest `version` is `4.9.3` (as of July 28, 2023), while the project uses version `4.8.3`.

That's why we strongly ``recommend`` that once the ``issues`` in the ``codebase`` identified by ``C4 Wardens`` are ``known``, the ``Ondo Finance team`` takes action to ``implement the mitigations`` and make their protocol a ``robust`` financial ecosystem for all ``participants``.

## 6- Monitoring Recommendations
While ``audits`` help in ``identifying`` code-level ``issues`` in the current implementation and potentially the code ``deployed`` in production, the ``Ondo Finance`` team is encouraged to consider incorporating monitoring activities in the production environment. Ongoing monitoring of deployed contracts helps identify potential threats and issues affecting production environments. With the goal of providing a complete ``security assessment``, the monitoring ``recommendations`` section raises several actions addressing trust assumptions and out-of-scope components that can benefit from ``on-chain monitoring``.

## 7- Financial activity
  - Consider monitoring the token transfers over the bridge to identify:
    * Transfers during normal operations to establish a baseline of healthy properties. Any large deviation, such as an ``unexpectedly`` large withdrawal, may indicate unusual ``behavior`` of the contracts or an ongoing attack.
    * Transactions that revert

These may indicate a ``user interface`` bug, an ongoing attack or other ``unexpected`` edge cases.
## Time Spent ⏱

A total of `3 days` were dedicated to completing this audit, distributed as follows:

1. 1st Day: Devoted to comprehending the protocol's functionalities and implementation.
2. 2nd Day: Initiated the analysis process, leveraging the documentation offered by the `Ondo Finance`.
3. 3rd Day: Focused on conducting a thorough ``analysis``, incorporating meticulously crafted ``diagrams`` derived from the ``contracts`` and ``information`` provided by the protocol.




### Time spent:
15 hours