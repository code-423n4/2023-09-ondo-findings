# Gas Optimization

# Summary

| Number | Issue                                                                                                                                              | Instances |
| :----: | :------------------------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant                                           |     5     |
| [G-02] | ++i COSTS LESS GAS THAN i++                                                                                                                        |     1     |
| [G-03] | Stack variable used as a cheaper cache for a state variable is only used once                                                                      |     2     |
| [G-04] | Can make the variable outside the loop to save Gas                                                                                                 |     2     |
| [G-05] | Require() statements should be used sorted from cheapest to most expensive                                                                         |    11     |
| [G-06] | Should use arguments instead of state variable                                                                                                     |     5     |
| [G-07] | Instead of calculating a STATEVAR with keccak256() every time the contract is made pre calclate them before and only give the result to a constant |     6     |
| [G-08] | Structs can be packed into fewer storage slots                                                                                                     |     1     |
| [G-09] | abi.encode() is less efficient than abi.encodePacked()                                                                                             |     3     |
| [G-10] | Do not calculate constants                                                                                                                         |     5     |
| [G-11] | Use do while loops instead of for loops                                                                                                            |     6     |
| [G-12] | Functions guaranteed to revert when called by normal users can be marked payable                                                                   |    27     |
| [G-13] | Use assembly to perform efficient back-to-back calls                                                                                               |     3     |
| [G-14] | missing zero address checks in the constructor                                                                                                     |     7     |
| [G-15] | Ternary operation is cheaper than if-else statement                                                                                                |     2     |
| [G-16] | Sort Solidity operations using short-circuit mode                                                                                                  |     1     |
| [G-17] | Use hardcode address instead address(this)                                                                                                         |     3     |
| [G-18] | use mappings instead of arrays                                                                                                                     |     1     |
| [G-19] | Shorten the array rather than copying to a new one                                                                                                 |     2     |
| [G-20] | Use uint256(1)/uint256(2 instead for true and false boolean states                                                                                 |     5     |
| [G-21] | Use Assembly To Check For address(0)                                                                                                               |     9     |

## [G-01] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

It's recommended to use immutable for constant values, such as the result of keccak256(), instead of constant. This change was introduced to improve gas efficiency and reduce deployment costs.

```solidity
File: contracts/usdy/rUSDY.sol

  bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
  bytes32 public constant LIST_CONFIGURER_ROLE =
    keccak256("LIST_CONFIGURER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97-L102

```solidity
File: rwaOracles/RWADynamicOracle.sol

27  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
28  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L27-L28

## [G-02] ++i COSTS LESS GAS THAN i++

In Solidity, ++i and i++ are both increment operations, but they have a slight difference in gas cost.

++i (pre-increment) increments the value of i and then returns the incremented value. It generally consumes slightly less gas compared to i++ because it doesn't need to create a temporary variable to store the original value of i.

i++ (post-increment) increments the value of i but returns the original value before the increment. It often consumes slightly more gas because it involves an additional step to store the original value in a temporary variable.

```solidity
File: bridge/SourceBridge.sol

79    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79-L80

## [G-03] Stack variable used as a cheaper cache for a state variable is only used once

```solidity
File: bridge/SourceBridge.sol

27  bytes32 public constant VERSION = "1.0";
```

This is state variable is used once in this function

```solidity

79    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L27-L27

## [G-04] Can make the variable outside the loop to save Gas

If you want to optimize gas costs in a loop, it's a good practice to declare variables outside the loop if they don't need to be reinitialized with each iteration. This can reduce unnecessary gas consumption from variable initialization within the loop.

```solidity
File: bridge/DestinationBridge.sol

135      Threshold memory t = thresholds[i];
```

```diff
diff --git a/org.sol b/not.sol
index 9caeee0..9a53e04 100644
--- a/org.sol
+++ b/not.sol
@@ -1,6 +1,7 @@
     Threshold[] memory thresholds = chainToThresholds[srcChain];
+    Threshold memory t;
     for (uint256 i = 0; i < thresholds.length; ++i) {
-      Threshold memory t = thresholds[i];
+       t = thresholds[i];
       if (amount <= t.amount) {
         txnToThresholdSet[txnHash] = TxnThreshold(
           t.numberOfApprovalsNeeded,
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L135-L136

```solidity
File: rwaOracles/RWADynamicOracle.sol

78      Range storage range = ranges[(length - 1) - i];
```

```diff
diff --git a/org.sol b/not.sol
index 03de3c7..0438d81 100644
--- a/org.sol
+++ b/not.sol
@@ -1,9 +1,10 @@
     uint256 length = ranges.length;
+    Range storage range;
     for (uint256 i = 0; i < length; ++i) {
-      Range storage range = ranges[(length - 1) - i];
+        range = ranges[(length - 1) - i];
       if (range.start <= block.timestamp) {
         if (range.end <= block.timestamp) {
           return derivePrice(range, range.end - 1);
         } else {
           return derivePrice(range, block.timestamp);
-        }

+        }

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L78-L79

## [G-05] Require() statements should be used sorted from cheapest to most expensive

While it's not strictly required to sort require() statements from cheapest to most expensive gas-wise, it can be a good practice to do so for readability and code organization. Sorting require() statements by gas cost can make it easier for other developers to understand the contract's logic and potential gas consumption. However, this is more of a convention than a strict rule.

```solidity
File: usdy/rUSDY.sol

    require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

    _beforeTokenTransfer(_account, address(0), _sharesAmount);

    uint256 accountShares = shares[_account];
    require(_sharesAmount <= accountShares, "BURN_AMOUNT_EXCEEDS_BALANCE");


    if (from != msg.sender && to != msg.sender) {
      require(!_isBlocked(msg.sender), "rUSDY: 'sender' address blocked");
      require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");
      require(
        _isAllowed(msg.sender),
        "rUSDY: 'sender' address not on allowlist"
      );
    }

    if (from != address(0)) {
      // If not minting
      require(!_isBlocked(from), "rUSDY: 'from' address blocked");
      require(!_isSanctioned(from), "rUSDY: 'from' address sanctioned");
      require(_isAllowed(from), "rUSDY: 'from' address not on allowlist");
    }

    if (to != address(0)) {
      // If not burning
      require(!_isBlocked(to), "rUSDY: 'to' address blocked");
      require(!_isSanctioned(to), "rUSDY: 'to' address sanctioned");
      require(_isAllowed(to), "rUSDY: 'to' address not on allowlist");
    }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L519-L528

## [G-06] Should use arguments instead of state variable

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol
// audit    ranges is state
164      emit RangeSet(
      ranges.length - 1,
      lastRange.end,
      endTimestamp,
      dailyInterestRate,
      prevClosePrice
    );
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L164-L170

```solidity
File:  contracts/usdy/rUSDYFactory.sol
101     emit rUSDYDeployed(
      address(rUSDYProxy),
      address(rUSDYProxyAdmin),
      address(rUSDYImplementation),
      "Ondo Rebasing U.S. Dollar Yield",
      "rUSDY"
    );
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L101-L107

## [G-07] Instead of calculating a STATEVAR with keccak256() every time the contract is made pre calclate them before and only give the result to a constant

```solidity
File: contracts/usdy/rUSDY.sol

  bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
  bytes32 public constant LIST_CONFIGURER_ROLE =
    keccak256("LIST_CONFIGURER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97-L102

```solidity
File: rwaOracles/RWADynamicOracle.sol

27  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
28  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L27-L28

## [G-08] Structs can be packed into fewer storage slots

```solidity
File: rwaOracles/RWADynamicOracle.sol

  struct Range {
    uint256 start;
    uint256 end;
    uint256 dailyInterestRate;
    uint256 prevRangeClosePrice;
  }
```

Save 2 Storage slots

```solidity
  struct Range {
    uint128 start;
    uint128 end;
    uint128 dailyInterestRate;
    uint128 prevRangeClosePrice;
  }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L295-L300

## [G-09] abi.encode() is less efficient than abi.encodePacked()

When gas efficiency is a concern, especially in scenarios where data will be used internally within the contract and won't be decoded externally, abi.encodePacked() is often preferred. It reduces unnecessary gas costs associated with including type information and results in a more compact data representation.

```solidity

79    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79

```solidity
File: bridge/DestinationBridge.sol

99    if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {

238    chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L99

## [G-10] Do not calculate constants

```solidity
File: contracts/usdy/rUSDY.sol

  bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
  bytes32 public constant LIST_CONFIGURER_ROLE =
    keccak256("LIST_CONFIGURER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97-L102

```solidity
File: rwaOracles/RWADynamicOracle.sol

27  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
28  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L27-L28

## [G-11] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops).

```solidity
File:  contracts/bridge/DestinationBridge.sol

159      if (t.approvers.length > 0) {
160      for (uint256 i = 0; i < t.approvers.length; ++i) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L159-L160

```solidity
File: bridge/SourceBridge.sol

164    for (uint256 i = 0; i < exCallData.length; ++i) {

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L164

```solidity
File: rwaOracles/RWADynamicOracle.sol

77   for (uint256 i = 0; i < length; ++i) {

113     for (uint256 i = 0; i < length; ++i) {


129    for (uint256 i = 0; i < length + 1; ++i) {

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

## [G-12] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File:  contracts/bridge/SourceBridge.sol
121     function setDestinationChainContractAddress(
    string memory destinationChain,
    address contractAddress
  ) external onlyOwner {

136  function pause() external onlyOwner {

145  function unpause() external onlyOwner {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L121

```solidity
File:  contracts/rwaOracles/RWADynamicOracle.sol
151    function setRange(
    uint256 endTimestamp,
    uint256 dailyInterestRate
  ) external onlyRole(SETTER_ROLE) {

186    function overrideRange(
    uint256 indexToModify,
    uint256 newStart,
    uint256 newEnd,
    uint256 newDailyIR,
    uint256 newPrevRangeClosePrice
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {

241 function pauseOracle() external onlyRole(PAUSER_ROLE) {

248  function unpauseOracle() external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L151

```solidity
File:   contracts/bridge/DestinationBridge.sol
210  function addApprover(address approver) external onlyOwner {

220  function removeApprover(address approver) external onlyOwner {

234    function addChainSupport(
    string calldata srcChain,
    string calldata srcContractAddress
  ) external onlyOwner {

255    function setThresholds(
    string calldata srcChain,
    uint256[] calldata amounts,
    uint256[] calldata numOfApprovers
  ) external onlyOwner {

286   function setMintLimit(uint256 mintLimit) external onlyOwner {

295  function setMintLimitDuration(uint256 mintDuration) external onlyOwner {

304  function pause() external onlyOwner {

313   function unpause() external onlyOwner {

322   function rescueTokens(address _token) external onlyOwner {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L210

```solidity
File:  contracts/usdy/rUSDYFactory.sol
81    ) external onlyGuardian returns (address, address, address) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L81

```solidity
File:  contracts/usdy/rUSDY.sol
120    function __rUSDY_init(
    address blocklist,
    address allowlist,
    address sanctionsList,
    address _usdy,
    address guardian,
    address _oracle
  ) internal onlyInitializing {

134    function __rUSDY_init_unchained(
    address _usdy,
    address guardian,
    address _oracle
  ) internal onlyInitializing {

662  function setOracle(address _oracle) external onlyRole(USDY_MANAGER_ROLE) {

685    function pause() external onlyRole(PAUSER_ROLE) {

689  function unpause() external onlyRole(USDY_MANAGER_ROLE) {

700  ) external override onlyRole(LIST_CONFIGURER_ROLE) {

711  ) external override onlyRole(LIST_CONFIGURER_ROLE) {

722  ) external override onlyRole(LIST_CONFIGURER_ROLE) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L120

## [G-13] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

```solidity
File:  contracts/bridge/DestinationBridge.sol

323      uint256 balance = IRWALike(_token).balanceOf(address(this));
324      IRWALike(_token).transfer(owner(), balance);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L324

## [G-14] missing zero address checks in the constructor

```solidity
File: bridge/SourceBridge.sol

    TOKEN = IRWALike(_token);
    AXELAR_GATEWAY = IAxelarGateway(_axelarGateway);
    GAS_RECEIVER = IAxelarGasService(_gasService);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L46-L48

```solidity
File:  bridge/DestinationBridge.sol

    TOKEN = IRWALike(_token);
    AXELAR_GATEWAY = IAxelarGateway(_axelarGateway);
    ALLOWLIST = IAllowlist(_allowlist);

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L67-L69

```solidity
File: usdy/rUSDYFactory.sol

  constructor(address _guardian) {
    guardian = _guardian;
  }

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L51-L54

## [G-15] Ternary operation is cheaper than if-else statement

```solidity
File:  bridge/DestinationBridge.sol

  if (t.numberOfApprovalsNeeded <= t.approvers.length) {
      return true;
    } else {
      return false;
    }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L179-L183

```solidity
File: rwaOracles/RWADynamicOracle.sol

      if (range.start <= block.timestamp) {
        if (range.end <= block.timestamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, block.timestamp);
        }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L79-L84

## [G-16] Sort Solidity operations using short-circuit mode

```solidity
File: rwaOracles/RWADynamicOracle.sol

405    require(y == 0 || (z = x * y) / y == x);

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L405

## [G-17] Use hardcode address instead address(this)

```solidity
File: bridge/SourceBridge.sol

97      address(this),

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L97

```solidity
File: bridge/DestinationBridge.sol

323    uint256 balance = IRWALike(_token).balanceOf(address(this));

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323

```solidity
File: usdy/rUSDY.sol

437    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L437

## [G-18] use mappings instead of arrays

Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity
File: rwaOracles/RWADynamicOracle.sol

25  Range[] public ranges;

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L25

## [G-19] Shorten the array rather than copying to a new one

```solidity
File: rwaOracles/RWADynamicOracle.sol

112    Range[] memory rangeList = new Range[](length + 1);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L112

```solidity
File: bridge/SourceBridge.sol

163    results = new bytes[](exCallData.length);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L163

## [G-20] Use uint256(1)/uint256(2 instead for true and false boolean states

```solidity
File: bridge/DestinationBridge.sol

106    isSpentNonce[chainToApprovedSender[srcChain]][nonce] = true;

180      return true;

182      return false;

211    approvers[approver] = true;

346          true
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L106

## [G-21] Use Assembly To Check For address(0)

it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

```solidity
File:  contracts/usdy/rUSDY.sol

490     require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");

491     require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");

519     require(_sender != address(0), "TRANSFER_FROM_THE_ZERO_ADDRESS");

520     require(_recipient != address(0), "TRANSFER_TO_THE_ZERO_ADDRESS");

547     require(_recipient != address(0), "MINT_TO_THE_ZERO_ADDRESS");

579     require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

642     if (from != address(0)) {

649     if (to != address(0)) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L490
