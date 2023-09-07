# Ondo Finance - Quality Assurance Report

# Low Risk Findings

## Table Of Content

| Number                                                                                                           | Issue                                                                                                | Instances |
| :--------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------- | --------: |
| [L-01](#l-01---contracts-should-use-the-latest-version-of-solidity-to-be-safe-from-bug-fixes)                    | Contracts should use the `latest version` of solidity to be safe from `bug fixes`                    |         6 |
| [L-02](#l-02---missing-validation-for-address0-in-the-constructors-or-in-the-functions-changing-state-variables) | Missing validation for `address(0)` in the constructors or in the functions changing state variables |        12 |
| [L-03](#l-03---shadow-variables)                                                                                 | Shadow Variables                                                                                     |         7 |
| [L-04](#l-04---unsafe-low-level-call)                                                                            | Unsafe `low-level` call                                                                              |         2 |
| [L-05](#l-05---division-by-zero-value-will-revert-the-transaction)                                               | `Division` by zero value will revert the transaction                                                 |         3 |
| [L-06](#l-06---multiplication-with-zero-value-will-make-the-whole-calculation-zero)                              | `Multiplication` with zero value will make the whole calculation zero                                |         6 |
| [L-07](#l-07---emitting-incorrect-event-information)                                                             | Emitting incorrect event information                                                                 |         1 |

### [L-01] - Contracts should use the `latest version` of solidity to be safe from `bug fixes`.

**Details**

The contracts in scope use the older version of solidity (`0.8.16`) and the current version while writing this report is (`0.8.21`).

Solidity docs always suggest using the latest version of Solidity for the deployment of smart contracts.

There are no breaking changes between these versions but numerous minor bug fixes have been done between them.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   pragma solidity 0.8.16;
+   pragma solidity 0.8.21;
```

`Findings Links`

[DestinationBridge.sol - Line 16](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L16)

[SourceBridge.sol - Line 01](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L1)

[IRWADynamicOracle.sol - Line 16](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/IRWADynamicOracle.sol#L16)

[RWADynamicOracle.sol - Line 16](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L16)

[rUSDYFactory.sol - Line 16](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L16)

[rUSDY.sol - Line 16](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L16)

</details>

**Recommended Mitigation Steps**

Update the contracts version to be safe from minor bug fixes between the used version and current solidity version.

### [L-02] - Missing validation for `address(0)` in the constructors or in the functions changing state variables.

**Details**

It is recommended to check for `address(0)` when updating state variables through constructor or functions. This can save the protocol from accidental updates.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```solidity
constructor(
    address _token,
    address _axelarGateway,
    address _allowlist,
    address _ondoApprover,
    address _owner,
    uint256 _mintLimit,
    uint256 _mintDuration
)
    AxelarExecutable(_axelarGateway)
    MintTimeBasedRateLimiter(_mintDuration, _mintLimit)
{
    require(
        _token != address(0) &&
        _axelarGateway != address(0) &&
        _allowlist != address(0) &&
        _ondoApprover != address(0) &&
        _owner != address(0),
        "DestinationBridge__InvalidAddress"
    );

    TOKEN = IRWALike(_token);
    AXELAR_GATEWAY = IAxelarGateway(_axelarGateway);
    ALLOWLIST = IAllowlist(_allowlist);
    approvers[_ondoApprover] = true;
    _transferOwnership(_owner);
}
```

`Findings Links`

[DestinationBridge.sol - Line 56 - 60](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L56-L60)

[DestinationBridge.sol - Line 210](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L210)

[DestinationBridge.sol - Line 220](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L220)

[SourceBridge.sol - Line 41 - 44](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L41-L44)

[RWADynamicOracle.sol - Line 31 - 33](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L31-L33)

[rUSDYFactory.sol - Line 52](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L52)

[rUSDYFactory.sol - Line 76 - 80](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L76-L80)

[rUSDY.sol - Line 110 - 115](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L110-L115)

[rUSDY.sol - Line 663](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L663)

[rUSDY.sol - Line 701](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L701)

[rUSDY.sol - Line 712](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L712)

[rUSDY.sol - Line 723](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L723)

</details>

### [L-03] - Shadow Variables.

**Details**

Consider the compiler warnings into consideration and avoid the variable shadowing.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   function setMintLimit(uint256 mintLimit) external onlyOwner {
+   function setMintLimit(uint256 _mintLimit) external onlyOwner {
-   _setMintLimit(mintLimit);
+   _setMintLimit(_mintLimit);
}
```

`Findings Links`

[DestinationBridge.sol - Line 286](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L286)

[SourceBridge.sol - Line 49](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L49)

[rUSDY.sol - Line 117](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L117)

[rUSDY.sol - Line 128 - 130](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L128-L130)

[rUSDY.sol - Line 701](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L701)

[rUSDY.sol - Line 712](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L712)

[rUSDY.sol - Line 723](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L723)

</details>

### [L-04] - Unsafe `low-level` call.

**Details**

```solidity
function multiexcall(
    ExCallData[] calldata exCallData
) external payable override onlyOwner returns (bytes[] memory results) {
    results = new bytes[](exCallData.length);

    for (uint256 i = 0; i < exCallData.length; ++i) {
        (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
        }(exCallData[i].data);

        require(success, "Call Failed");

        results[i] = ret;
    }
}
```

Solidity docs warn about using low-level calls directly due to security reasons;

-   You should avoid using `.call()` whenever possible when executing another contract function as it bypasses type checking, function existence check, and argument packing.

-   Due to the fact that the EVM considers a call to a non-existing contract to always succeed, Solidity includes an extra check using the `extcodesize` opcode when performing external calls. This ensures that the contract that is about to be called either actually exists (it contains code) or an exception is raised. The low-level calls which operate on addresses rather than contract instances (i.e. `.call()`, `.delegatecall()`, `.staticcall()`, `.send()` and `.transfer()`) **do not** include this check, which makes them cheaper in terms of gas but also less safe.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[SourceBridge.sol - Line 165](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L165)

[rUSDYFactory.sol - Line 131](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L131)

</details>

**Recommended Mitigation Steps**

Use Openzeppelin [Address](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol) library to perform the external call.

### [L-05] - `Division` by zero value will revert the transaction.

**Details**

Below are the mentioned findings where the denominator is not checked for zero value and these conditions will always cause revert.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RWADynamicOracle.sol - Line 44](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L44)

[RWADynamicOracle.sol - Line 117](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L117)

[RWADynamicOracle.sol - Line 219](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L219)

</details>

### [L-06] - `Multiplication` with zero value will make the whole calculation zero.

**Details**

Below are the mentioned findings where the values passed through arguments are not checked for zero value and these can make the whole calculation zero.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RWADynamicOracle.sol - Line 44](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L44)

[RWADynamicOracle.sol - Line 117](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L117)

[RWADynamicOracle.sol - Line 219](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L219)

```solidity
function getPrice() public view whenNotPaused returns (uint256 price) {
    uint256 length = ranges.length;

    for (uint256 i = 0; i < length; ++i) {
        Range storage range = ranges[(length - 1) - i];

        if (range.start <= block.timestamp) {
        if (range.end <= block.timestamp) {
            return derivePrice(range, range.end - 1);
        } else {
            return derivePrice(range, block.timestamp);
        }
        }
    }
}
```

In this function, if the conditions do not meet then the return price can be `0`. Below are the links where this function is used.

[rUSDY.sol - Line 217](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L217)

[rUSDY.sol - Line 227](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L227)

[rUSDY.sol - Line 398](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L398)

</details>

### [L-07] - Emitting incorrect event information.

**Details**

Correct event information is necessary to facilitate off-chain searching and filtering for specific events information by applications. Emitting wrong information can cause effect there.

Below are the mentioned spots where emitted information is incorrect.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

```diff
-   emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
+   emit Transfer(address(0), msg.sender, _USDYAmount);
-   emit TransferShares(address(0), msg.sender, _USDYAmount);
+   emit TransferShares(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
```

[rUSDY.sol - Line 438 - 439](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L438-L439)

</details>

# Non Critical Findings

## Table Of Content

| Number                                                                                                                                       | Issue                                                                                                                             | Instances |
| :------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- | --------- |
| [I-01](#i-01---always-import-other-source-files-explicitly)                                                                                  | Always `import` other source files explicitly                                                                                     | 5         |
| [I-02](#i-02---use-natspec-comments-for-smart-contracts-interfaces-and-libraries)                                                            | Use `Natspec` comments for smart contracts, interfaces and libraries                                                              | 4         |
| [I-03](#i-03---set-the-internal-layout-of-contracts-interfaces-and-libraries)                                                                | Set the Internal Layout of Contracts, Interfaces, and Libraries                                                                   | 4         |
| [I-04](#i-04---lack-of-indexed-events-parameters)                                                                                            | Lack of `indexed` events parameters                                                                                               | 2         |
| [I-05](#i-05---license-information-missing-in-the-contract)                                                                                  | License information missing in the contract                                                                                       | 1         |
| [I-06](#i-06---use-natspec-comments-for-events-state-variables-constructor-functions-and-all-other-things-which-will-be-included-in-the-abi) | Use `Natspec` comments for events, state variables, constructor, functions and all other things which will be included in the ABI | 1         |

### [I-01] - Always `import` other source files explicitly.

**Details**

Soldity docs suggest to always import explicitly because just import will import all of the things from the file.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   import "contracts/interfaces/IAxelarGateway.sol";
+   import { IAxelarGateway } from "contracts/interfaces/IAxelarGateway.sol";
```

`Findings Links`

[DestinationBridge.sol - Line 18 - 25](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L18-L25)

[SourceBridge.sol - Line 03 - 09](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L3-L9)

[RWADynamicOracle.sol - Line 18 - 20](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L18-L20)

[rUSDYFactory.sol - Line 19 - 22](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L19-L22)

[rUSDY.sol - Line 18 - 28](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L18-L28)

</details>

### [I-02] - Use `Natspec` comments for smart contracts, interfaces and libraries.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[DestinationBridge.sol - Line 27](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L27)

[SourceBridge.sol - Line 11](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L11)

[IRWADynamicOracle.sol - Line 18](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/IRWADynamicOracle.sol#L18)

[rUSDYFactory.sol - Line 44 - 157](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L44-L157)

</details>

### [I-03] - Set the Internal Layout of Contracts, Interfaces, and Libraries.

**Details**

According to the Solidity [Style Guide](https://docs.soliditylang.org/en/v0.8.21/style-guide.html#style-guide) consistency in the projects layout is very important. If all projects apply the style guides provided by solidity docs then understanding the projects and its code will be much easier for developers, and auditors.

Solidity docs;

> A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is most important.

According to the solidity styles guide;

Inside each contract, library or interface, this order should be followed:

1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

And this should be the order of functions:

-   constructor
-   receive function (if exists)
-   fallback function (if exists)
-   external
-   public
-   internal
-   private
-   View and pure functions last

Apply the recommeded layout sturcture in all mentioned contracts according to solidity styles guide.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[DestinationBridge.sol - Line 33 - 448](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L33-L448)

[SourceBridge.sol - Line 12 - 190](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L12-L190)

[RWADynamicOracle.sol - Line 23 - 406](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L23-L406)

[rUSDYFactory.sol - Line 44 - 157](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L44-L157)

</details>

### [I-04] - Lack of `indexed` events parameters.

**Details**

To facilitate off-chain searching and filtering for specific events information try to utilize the 3 available `indexed` parameters for events.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
/**
* @notice event emitted when an address is removed as an approver
*
* @param approver The address being removed
*/
-   event ApproverRemoved(address approver);
+   event ApproverRemoved(address indexed approver);
```

`Findings Links`

[DestinationBridge.sol - Line 389 - 437](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L389-L437)

[rUSDYFactory.sol - Line 146 - 152](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L146-L152)

</details>

### [I-05] - License information missing in the contract.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

```diff
+   // SPDX-License-Identifier: BUSL-1.1
```

[SourceBridge.sol - Line 01](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L1)

</details>

### [I-06] - Use `Natspec` comments for events, state variables, constructor, functions and all other things which will be included in the ABI.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RWADynamicOracle.sol - Line 23 - 406](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L23-L406)

</details>