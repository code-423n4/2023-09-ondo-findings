|     |
| --- |
| \[G-01\] abi.encode() is less efficient than abi.encodePacked() |
| \[G-02\] Events are not indexed |
| \[G-03\] Make 3 event parameters indexed when possible |
| \[G-04\] Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant |
| \[G-05\] Do not calculate constants |
| \[G-06\]  `10 ** 27` can be changed to `1e27` and save some gas |
| \[G‑07\] Avoid contract existence checks by using low level calls |
| \[G-08\] Make fewer external calls. |
| \[G-09\] Use assembly for math (add, sub, mul, div) |

## \[G-01\] abi.encode() is less efficient than abi.encodePacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost.

```
bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L79

## <ins>\[G-02\] Events are not indexed</ins>

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

```
event ApproverRemoved(address approver);

  /**
   * @notice event emitted when an address is added as an approver
   *
   * @param approver  The address to add
   */
  event ApproverAdded(address approver);

  /**
   * @notice event emitted when a new contract is whitelisted as an approved
   *         message passer.
   *
   * @param srcChain        The chain for the approved address
   * @param approvedSource  The address corresponding to the source bridge contract
   */
  event ChainIdSupported(string srcChain, string approvedSource);

  /**
   * @notice event emitted when a threshold has been set
   *
   * @param chain           The chain for which the threshold was set
   * @param amounts         The amount of tokens to reach this threshold
   * @param numOfApprovers  The number of approvals needed
   */
  event ThresholdSet(string chain, uint256[] amounts, uint256[] numOfApprovers);

  /**
   * @notice event emitted when the user has been minted their tokens on the dst chain
   *
   * @param user    The recipient address of the newly minted tokens
   * @param amount  The amount of tokens that have been minted
   */
  event BridgeCompleted(address user, uint256 amount);
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L389C2-L423C1

```
event MessageReceived(
    string srcChain,
    address srcSender,
    uint256 amt,
    uint256 nonce
  );
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L432C3-L437C5

```
event RangeSet(
    uint256 indexed index,
    uint256 start,
    uint256 end,
    uint256 dailyInterestRate,
    uint256 prevRangeClosePrice
  );

  /**
   * @notice Event emitted when a previously set range is overriden
   *
   * @param index                  The index of the range being modified
   * @param newStart               The new start time for the modified range
   * @param newEnd                 The new end time for the modified range
   * @param newDailyInterestRate   The new dailyInterestRate for the modified range
   * @param newPrevRangeClosePrice The new prevRangeClosePrice for the modified range
   */
  event RangeOverriden(
    uint256 indexed index,
    uint256 newStart,
    uint256 newEnd,
    uint256 newDailyInterestRate,
    uint256 newPrevRangeClosePrice
  );
```

## https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L309C1-L332C5

## \[G-03\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

[Reference](https://code4rena.com/reports/2023-01-drips#g-05-make-3-event-parameters-indexed-when-possible)

```
event ApproverRemoved(address approver);

  /**
   * @notice event emitted when an address is added as an approver
   *
   * @param approver  The address to add
   */
  event ApproverAdded(address approver);

  /**
   * @notice event emitted when a new contract is whitelisted as an approved
   *         message passer.
   *
   * @param srcChain        The chain for the approved address
   * @param approvedSource  The address corresponding to the source bridge contract
   */
  event ChainIdSupported(string srcChain, string approvedSource);
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L389C2-L423C1

```
event BridgeCompleted(address user, uint256 amount);
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L422

## \[G-04\] Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant

```
bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
  bytes32 public constant LIST_CONFIGURER_ROLE =
    keccak256("LIST_CONFIGURER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L97C2-L102C39

```
bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L27C3-L29C1

## \[G-05\] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```
uint256 private constant ONE = 10 ** 27;
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L343

## \[G-06\]  `10 ** 27` can be changed to `1e27` and save some gas

`10 ** 27` can be changed to `1e27` to avoid unnecessary arithmetic operation and save gas.

```
uint256 private constant ONE = 10 ** 27;
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L343

## \[G‑07\] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

## \[G-08\] Make fewer external calls.

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, It’s better to call one function and have it return all the data you need rather than calling a separate function for every piece of data. This might go against the best coding practices for other languages, but solidity is special.