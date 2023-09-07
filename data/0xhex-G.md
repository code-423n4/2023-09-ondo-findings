# Gas Optimization

# Summary

| Number | Gas                                                                                                      | context |
| :----: | :------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Functions guaranteed to revert when called by normal users can be marked payable                         |   25    |
| [G-02] | Use assembly to perform efficient back-to-back calls                                                     |    2    |
| [G-03] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant |    5    |
| [G-04] | Can make the variable outside the loop to save Gas                                                       |    2    |
| [G-05] | Avoid emitting storage values                                                                            |    2    |
| [G-06] | Avoid contract existence checks by using low level calls                                                 |    3    |
| [G-07] | Use do while loops instead of for loops                                                                  |    5    |
| [G-08] | abi.encode() is less efficient than abi.encodePacked()                                                   |    3    |
| [G-09] | Stack variable used as a cheaper cache for a state variable is only used once                            |    1    |
| [G-10] | Use Assembly To Check For address(0)                                                                     |    8    |

## [G-01] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

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

## [G-02] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

```solidity
File:  contracts/bridge/DestinationBridge.sol


323      uint256 balance = IRWALike(_token).balanceOf(address(this));

324      IRWALike(_token).transfer(owner(), balance);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L324

## [G-03] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

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

## [G-04] Can make the variable outside the loop to save Gas

If you want to optimize gas costs in a loop, it's a good practice to declare variables outside the loop if they don't need to be reinitialized with each iteration. This can reduce unnecessary gas consumption from variable initialization within the loop.

```solidity
File: bridge/DestinationBridge.sol

135      Threshold memory t = thresholds[i];
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L135-L136

```solidity
File: rwaOracles/RWADynamicOracle.sol

78      Range storage range = ranges[(length - 1) - i];
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L78-L79

## [G-05] Avoid emitting storage values

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.
Avoid emitting storage values

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

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

## [G-06] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File:  contracts/bridge/DestinationBridge.sol

323      uint256 balance = IRWALike(_token).balanceOf(address(this));

324      IRWALike(_token).transfer(owner(), balance);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L324

## [G-07] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops).

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

## [G-08] abi.encode() is less efficient than abi.encodePacked()

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

## [G-09] Stack variable used as a cheaper cache for a state variable is only used once

```solidity
File: bridge/SourceBridge.sol

27  bytes32 public constant VERSION = "1.0";
```

## [G-10] Use Assembly To Check For address(0)

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