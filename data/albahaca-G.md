# gas 

# summary

|      | issue | instance |
|------|-------|----------|
|[G-01]| Use Assembly To Check For address(0)|6|
|[G-02]| Avoid contract existence checks by using low level calls |2|
|[G-03]| Use do while loops instead of for loops|2|
|[G-04]| Functions guaranteed to revert when called by normal users can be marked payable|35|
|[G-05]| Avoid emitting storage values |2|
|[G-06]| Use assembly to perform efficient back-to-back calls|2|


## [G-01] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

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

## [G-02] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File:  contracts/bridge/DestinationBridge.sol
323      uint256 balance = IRWALike(_token).balanceOf(address(this));

324      IRWALike(_token).transfer(owner(), balance);
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L324


## [G-03] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)

```solidity
File:  contracts/bridge/DestinationBridge.sol
159      if (t.approvers.length > 0) {
160      for (uint256 i = 0; i < t.approvers.length; ++i) {
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L159-L160

## [G-04] Functions guaranteed to revert when called by normal users can be marked payable
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

```solidity
File:  contracts/usdy/rUSDYFactory.sol
81    ) external onlyGuardian returns (address, address, address) {    
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L81

## [G-05] Avoid emitting storage values
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

## [G-06] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

```solidity
File:  contracts/bridge/DestinationBridge.sol
323      uint256 balance = IRWALike(_token).balanceOf(address(this));
324      IRWALike(_token).transfer(owner(), balance);
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323-L324