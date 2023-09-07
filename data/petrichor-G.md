# gas 

# summary

|       | issue | instance |
|-------|-------|----------|
|[G-01]|Functions guaranteed to revert when called by normal users can be marked payable|25|
|[G-02]|Use assembly to perform efficient back-to-back calls|2|
|[G-03]|Use assembly to perform efficient back-to-back calls|2|
|[G-04]|Avoid emitting storage values|2|
|[G-05]|Use Assembly To Check For address(0)|8|
|[G-06]|Use do while loops instead of for loops|2|
|[G-07]|not equal to zero" (!= 0) or the "equal to zero" (== 0)|1|


## [G-01] Functions guaranteed to revert when called by normal users can be marked payable


If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

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
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L220
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L234
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L255
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L286
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L295
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L304
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L313
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L322



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
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L136

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L145


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
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L186
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L241
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L248


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
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L134
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L662
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L685
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L689
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L700
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L711
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L722


```solidity
File:  contracts/usdy/rUSDYFactory.sol
81    ) external onlyGuardian returns (address, address, address) {    
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L81



## [G-02] Use assembly to perform efficient back-to-back calls

 we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity
File:  contracts/bridge/DestinationBridge.sol
323      uint256 balance = IRWALike(_token).balanceOf(address(this));
324      IRWALike(_token).transfer(owner(), balance);
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L323
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L324




## [G-03] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File:  contracts/bridge/DestinationBridge.sol
323      uint256 balance = IRWALike(_token).balanceOf(address(this));

324      IRWALike(_token).transfer(owner(), balance);
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L323
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L324


## [G-04] Avoid emitting storage values
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


## [G-05] Use Assembly To Check For address(0)

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary


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
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L491
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L519
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L520
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L547
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L579
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L642
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L649




## [G-06] Use do while loops instead of for loops
A do while loop is a post-condition loop, which means that the loop body is executed at least once before checking the loop condition. This can be advantageous when you want to avoid unnecessary iterations and save gas.


```solidity
File:  contracts/bridge/DestinationBridge.sol
159      if (t.approvers.length > 0) {
160      for (uint256 i = 0; i < t.approvers.length; ++i) {
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L159-L159
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L159-L160


## [G-7]  not equal to zero" (!= 0) or the "equal to zero" (== 0)

In Solidity, when comparing unsigned integers to zero, you can use either the "not equal to zero" (!= 0) or the "equal to zero" (== 0) comparison operators interchangeably. Both operators serve the purpose of evaluating whether an unsigned integer holds a non-zero or zero value.


```solidity
159    if (t.approvers.length > 0) {
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L159
