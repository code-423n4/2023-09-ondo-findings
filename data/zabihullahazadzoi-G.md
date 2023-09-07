# Ondo Finance - Gas Optimizations Report

**Notes**: 
- For Gas estimates I’ve tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average, before and after will be included, and often a diff of the code will also accompany this.
Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the opcodes involved.
- Instances pointed out in G-04, G-05, G-08, have missed by bot race and aren't included in automated report.


# Summary

|   Number   |  Issue   |  Instances  |
|------|----------|------------|
|[G-01]| Use calldata instead of memory for function arguments that do not get mutated|2|
|[G-02]| Access mappings directly rather than using accessor functions|1|
|[G-03]| Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|8|
|[G-04]| >=/<= costs less gas than >/<|4|
|[G-05]| Amounts should be checked for 0 before calling a transfer|1|
|[G-06]| Use hardcode address instead address(this)|3|
|[G-07]| Use Assembly To Check For address(0)|8|
|[G-08]| Use assembly to hash instead of Solidity|6|
|[G-09]| ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for loops|8|



## [G-01] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs as well as cost more gas for protocol.

Gas saving for `SourceBridge.setDestinationChainContractAddress`, obtained via protocol's tests: Avg 394 gas.

| | Min| Average| Max  |
|------|----------|-------|-----|
|Before|3035|83198|90409|
|After|2894|82804|89998|


```solidity
File:	contracts/bridge/SourceBridge.sol
122         string memory destinationChain,
```

```diff
diff --git a/contracts/bridge/SourceBridge.sol b/contracts/bridge/SourceBridge.sol
index 457bf41..853bcaf 100644
--- a/contracts/bridge/SourceBridge.sol
+++ b/contracts/bridge/SourceBridge.sol
@@ -119,7 +119,7 @@ contract SourceBridge is Ownable, Pausable, IMulticall {
    *      at https://docs.axelar.dev/dev/reference/mainnet-chain-names
    */
   function setDestinationChainContractAddress(
-    string memory destinationChain,
+    string calldata destinationChain,
     address contractAddress
   ) external onlyOwner {
     destChainToContractAddr[destinationChain] = AddressToString.toString(
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L122

```solidity
File:	contracts/bridge/DestinationBridge.sol
131         string memory srcChain
```

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index 8ad410c..fd43f6b 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -128,7 +128,7 @@ contract DestinationBridge is
   function _attachThreshold(
     uint256 amount,
     bytes32 txnHash,
-    string memory srcChain
+    string calldata srcChain
   ) internal {
     Threshold[] memory thresholds = chainToThresholds[srcChain];
     for (uint256 i = 0; i < thresholds.length; ++i) {
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L131


## [G-02] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup.

Gas saving for `rUSDY.balanceOf`, obtained via protocol's tests: Avg 103 gas.
Note: this contract can be optimized further more by removing all those accessor functions that are no more needed.

| | Min| Average| Max  |
|------|----------|-------|-----|
|Before|1727|2708|5084|
|After|1726|2605|5202|


```solidity
File:	contracts/usdy/rUSDY.sol
227         return (_sharesOf(_account) * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
```

```diff
diff --git a/contracts/usdy/rUSDY.sol b/contracts/usdy/rUSDY.sol
index 943da0d..7df8ff8 100644
--- a/contracts/usdy/rUSDY.sol
+++ b/contracts/usdy/rUSDY.sol
@@ -224,7 +224,7 @@ contract rUSDY is > 
    *      by the price of USDY
    */
   function balanceOf(address _account) public view returns (uint256) {
-    return (_sharesOf(_account) * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
+    return (shares[_account] * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
   }
 
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L227


## [G-03] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.
In contrast, immutable variables are evaluated at compilation time, and their values are included in the bytecode of the contract as constants. This means that any expensive operations performed as part of the immutable expression are only executed once, when the contract is compiled, and the result is reused every time the contract is deployed. This can result in lower gas costs compared to using constant variables.

```solidity
File:	contracts/rwaOracles/RWADynamicOracle.sol
27		bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
28		bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
343		uint256 private constant ONE = 10 ** 27;
```

```diff
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index 03aa7d6..6b5999f 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -24,8 +24,8 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
 
   Range[] public ranges;
 
-  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
-  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
+  bytes32 public immutable SETTER_ROLE = keccak256("SETTER_ROLE");
+  bytes32 public immutable PAUSER_ROLE = keccak256("PAUSER_ROLE");
 
   constructor(
     address admin,
@@ -340,7 +340,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
   //////////////////////////////////////////////////////////////*/
 
   // Copied from https://github.com/makerdao/dss/blob/master/src/jug.sol
-  uint256 private constant ONE = 10 ** 27;
+  uint256 private constant immutable ONE = 10 ** 27;
 
   function _rpow(
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L27
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L28
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L343


```solidity
File:	contracts/usdy/rUSDY.sol
97      bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
98	bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
99	bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
100	bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
101	bytes32 public constant LIST_CONFIGURER_ROLE =
```

```diff
diff --git a/contracts/usdy/rUSDY.sol b/contracts/usdy/rUSDY.sol
index 7df8ff8..515e73e 100644
--- a/contracts/usdy/rUSDY.sol
+++ b/contracts/usdy/rUSDY.sol
@@ -94,11 +94,11 @@ contract rUSDY is
   error UnwrapTooSmall();
 
   /// @dev Role based access control roles
-  bytes32 public USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
-  bytes32 public MINTER_ROLE = keccak256("MINTER_ROLE");
-  bytes32 public PAUSER_ROLE = keccak256("PAUSER_ROLE");
-  bytes32 public BURNER_ROLE = keccak256("BURN_ROLE");
-  bytes32 public LIST_CONFIGURER_ROLE =
+  bytes32 public immutable USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
+  bytes32 public immutable MINTER_ROLE = keccak256("MINTER_ROLE");
+  bytes32 public immutable PAUSER_ROLE = keccak256("PAUSER_ROLE");
+  bytes32 public immutable BURNER_ROLE = keccak256("BURN_ROLE");
+  bytes32 public immutable LIST_CONFIGURER_ROLE =
     keccak256("LIST_CONFIGURER_ROLE");
 
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L97


## [G-04] >=/<= costs less gas than >/<
The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=, and it can save 3 gas for each.

Note: These instances were missed by bot race and weren't included in Automated report.

Total new instances: 4
Gas saved: 4 * 3 = 12


```solidity
File:	contracts/rwaOracles/RWADynamicOracle.sol
207	if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();

212	if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();

214	 if (newEnd > ranges[indexToModify + 1].start) revert InvalidRange();
```

```diff
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index de82918..b60260a 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -204,14 +204,14 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
     // Case 2: The range being modified is the last range
     else if (indexToModify == rangeLength - 1) {
       // Ensure that the newStart time is not less than the end time of the previous range
-      if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();
+      if (newStart <= ranges[indexToModify - 1].end) revert InvalidRange();
     }
     // Case 3: The range being modified is between first and last range
     else {
       // Ensure that the newStart time is less than the end time of the previous range
-      if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();
+      if (newStart <= ranges[indexToModify - 1].end) revert InvalidRange();
       // Ensure that the newEnd time is not greater than the start time of the next range
-      if (newEnd > ranges[indexToModify + 1].start) revert InvalidRange();
+      if (newEnd >= ranges[indexToModify + 1].start) revert InvalidRange();
     }
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L207
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L212
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L214

```solidity
File:	contracts/bridge/DestinationBridge.sol
270	 if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
```

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index fd43f6b..79c7cf8 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -267,7 +267,7 @@ contract DestinationBridge is
           Threshold(amounts[i], numOfApprovers[i])
         );
       } else {
-        if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
+        if (chainToThresholds[srcChain][i - 1].amount >= amounts[i]) {
           revert ThresholdsNotInAscendingOrder();
         }
         chainToThresholds[srcChain].push(
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L270


## [G-05] Amounts should be checked for 0 before calling a transfer
In Solidity, unnecessary operations can waste gas. For example, a transfer function without a zero amount check uses gas even if called with a zero amount, since the contract state remains unchanged. Implementing a zero amount check avoids these unnecessary function calls, saving gas and improving efficiency.

Note: Instances below were missed from the bot race.


```solidity
File:	contracts/usdy/rUSDY.sol
680         usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);
```

```diff
diff --git a/contracts/usdy/rUSDY.sol b/contracts/usdy/rUSDY.sol
index 8bee380..f2b95b4 100644
--- a/contracts/usdy/rUSDY.sol
+++ b/contracts/usdy/rUSDY.sol
@@ -673,6 +673,7 @@ contract rUSDY is
     address _account,
     uint256 _amount
   ) external onlyRole(BURNER_ROLE) {
+    require(_amount > 0, "rUSDY: can't burn zero rUSDY tokens");
     uint256 sharesAmount = getSharesByRUSDY(_amount);
 
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L680


## [G-06] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

```solidity
File:	contracts/bridge/DestinationBridge.sol
323	uint256 balance = IRWALike(_token).balanceOf(address(this));
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L323

```solidity
File:	contracts/bridge/SourceBridge.sol
97	address(this),
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L97

```solidity
File:	contracts/usdy/rUSDY.sol
437	usdy.transferFrom(msg.sender, address(this), _USDYAmount);
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L437


## [G-07] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == or != operators.

The reason for this is that the == or != operators generate additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

```solidity
File:	contracts/usdy/rUSDY.sol
490	require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");

491	require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");

519	require(_sender != address(0), "TRANSFER_FROM_THE_ZERO_ADDRESS");

520	require(_recipient != address(0), "TRANSFER_TO_THE_ZERO_ADDRESS");

547	require(_recipient != address(0), "MINT_TO_THE_ZERO_ADDRESS");

579	require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

642	if (from != address(0)) {

649	if (to != address(0)) {

```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L490


## [G-08] Use assembly to compute hashes to save gas
If the arguments to the encode call can fit into the scratch space (two words or fewer), then it's more efficient to use assembly to generate the hash (80 gas).

Note: Instances below were missed by bot race and aren't included in automated report.

Total new instances: 6
Gas saved: 6 * 80 = 480


```solidity
File:	contracts/rwaOracles/RWADynamicOracle.sol
27	bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");

28	bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L27
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L28


```solidity
File:	contracts/usdy/rUSDY.sol
97	bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");

98	bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

99	bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

100	bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");

```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L97


## [G-09] U++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop iteration

Gas saving for `DestinationBridge.setThresholds`, obtained via protocol's tests: Avg 115 gas.

| | Min| Average| Max  |
|------|----------|-------|-----|
|Before|3210|105454|118448|
|After|3210|105339|118322|


```solidity
File:	contracts/bridge/DestinationBridge.sol
264           for (uint256 i = 0; i < amounts.length; ++i) {
```

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index 79c7cf8..f4d91bc 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -45,7 +45,7 @@ contract DestinationBridge is

   /// @notice Mappings used to track transaction and thresholds
   mapping(bytes32 => TxnThreshold) public txnToThresholdSet;
@@ -261,7 +261,7 @@ contract DestinationBridge is
       revert ArrayLengthMismatch();
     }
     delete chainToThresholds[srcChain];
-    for (uint256 i = 0; i < amounts.length; ++i) {
+    for (uint256 i = 0; i < amounts.length; ) {
       if (i == 0) {
         chainToThresholds[srcChain].push(
           Threshold(amounts[i], numOfApprovers[i])
@@ -274,6 +274,9 @@ contract DestinationBridge is
           Threshold(amounts[i], numOfApprovers[i])
         );
       }
+      unchecked {
+        ++i;
+      }
     }
     emit ThresholdSet(srcChain, amounts, numOfApprovers);
   }
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L264

```solidity
File:	contracts/bridge/DestinationBridge.sol
134     for (uint256 i = 0; i < thresholds.length; ++i) {

160	for (uint256 i = 0; i < t.approvers.length; ++i) {
```

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index f4d91bc..65b47e4 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -131,7 +131,7 @@ contract DestinationBridge is
     string calldata srcChain
   ) internal {
     Threshold[] memory thresholds = chainToThresholds[srcChain];
-    for (uint256 i = 0; i < thresholds.length; ++i) {
+    for (uint256 i = 0; i < thresholds.length; ) {
       Threshold memory t = thresholds[i];
       if (amount <= t.amount) {
         txnToThresholdSet[txnHash] = TxnThreshold(
@@ -140,6 +140,9 @@ contract DestinationBridge is
         );
         break;
       }
+      unchecked {
+        ++i;
+      }
     }
     if (txnToThresholdSet[txnHash].numberOfApprovalsNeeded == 0) {
       revert NoThresholdMatch();
@@ -157,10 +160,14 @@ contract DestinationBridge is
     // Check that the approver has not already approved
     TxnThreshold storage t = txnToThresholdSet[txnHash];
     if (t.approvers.length > 0) {
-      for (uint256 i = 0; i < t.approvers.length; ++i) {
+      for (uint256 i = 0; i < t.approvers.length; ) {
         if (t.approvers[i] == msg.sender) {
           revert AlreadyApproved();
         }
+
+        unchecked {
+          ++i;
+        }
       }
     }
     t.approvers.push(msg.sender);
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L134
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L160


Gas saving for `SourceBridge.multiexcall`, obtained via protocol's tests: Avg 25 gas.

| | Min| Average| Max  |
|------|----------|-------|-----|
|Before|2719|16677|30635|
|After|2719|16652|30585|


```solidity
File:	contracts/bridge/DestinationBridge.sol
264           for (uint256 i = 0; i < amounts.length; ++i) {
```

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index 79c7cf8..f4d91bc 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -45,7 +45,7 @@ contract DestinationBridge is

   /// @notice Mappings used to track transaction and thresholds
   mapping(bytes32 => TxnThreshold) public txnToThresholdSet;
@@ -261,7 +261,7 @@ contract DestinationBridge is
       revert ArrayLengthMismatch();
     }
     delete chainToThresholds[srcChain];
-    for (uint256 i = 0; i < amounts.length; ++i) {
+    for (uint256 i = 0; i < amounts.length; ) {
       if (i == 0) {
         chainToThresholds[srcChain].push(
           Threshold(amounts[i], numOfApprovers[i])
@@ -274,6 +274,9 @@ contract DestinationBridge is
           Threshold(amounts[i], numOfApprovers[i])
         );
       }
+      unchecked {
+        ++i;
+      }
     }
     emit ThresholdSet(srcChain, amounts, numOfApprovers);
   }
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L264

Gas saving for `RWADynamicOracle.getPrice`, obtained via protocol's tests: Avg 90 gas.

| | Min| Average| Max  |
|------|----------|-------|-----|
|Before|490|4016|14829|
|After|473|3926|14739|


```solidity
File:	contracts/rwaOracles/RWADynamicOracle.sol
77       for (uint256 i = 0; i < length; ++i) {
```

```diff
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index 4943d11..d08c27e 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -74,7 +74,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
    */
   function getPrice() public view whenNotPaused returns (uint256 price) {
     uint256 length = ranges.length;
-    for (uint256 i = 0; i < length; ++i) {
+    for (uint256 i = 0; i < length; ) {
       Range storage range = ranges[(length - 1) - i];
       if (range.start <= block.timestamp) {
         if (range.end <= block.timestamp) {
@@ -83,6 +83,9 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
           return derivePrice(range, block.timestamp);
         }
       }
+      unchecked {
+        ++i;
+      }
     }
   }
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L77

Gas saving for `RWADynamicOracle.simulateRange`, obtained via protocol's tests: Avg 202 gas.

| | Min| Average| Max  |
|------|----------|-------|-----|
|Before|4109|7319|17119|
|After|4047|7117|17057|


```solidity
File:	contracts/rwaOracles/RWADynamicOracle.sol
113      for (uint256 i = 0; i < length; ++i) {

129	 for (uint256 i = 0; i < length + 1; ++i) {
```

```diff
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index d08c27e..2d56b04 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -113,8 +113,12 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
   ) external view returns (uint256 price) {
     uint256 length = ranges.length;
     Range[] memory rangeList = new Range[](length + 1);
-    for (uint256 i = 0; i < length; ++i) {
+    for (uint256 i = 0; i < length; ) {
       rangeList[i] = ranges[i];
+
+      unchecked {
+        ++i;
+      }
     }
     if (startTime == ranges[0].start) {
       uint256 trueStart = (rangeStartPrice * ONE) / dailyIR;
@@ -129,7 +133,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
         prevClosePrice
       );
     }
-    for (uint256 i = 0; i < length + 1; ++i) {
+    for (uint256 i = 0; i < length + 1; ) {
       Range memory range = rangeList[(length) - i];
       if (range.start <= blockTimeStamp) {
         if (range.end <= blockTimeStamp) {
@@ -138,6 +142,9 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
           return derivePrice(range, blockTimeStamp);
         }
       }
+      unchecked {
+        ++i;
+      }
     }
   }
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L113
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L129

```solidity
File:	contracts/usdy/rUSDYFactory.sol
130      for (uint256 i = 0; i < exCallData.length; ++i) {
```

```diff
diff --git a/contracts/usdy/rUSDYFactory.sol b/contracts/usdy/rUSDYFactory.sol
index 62178a5..9ad9ce3 100644
--- a/contracts/usdy/rUSDYFactory.sol
+++ b/contracts/usdy/rUSDYFactory.sol
@@ -127,12 +127,16 @@ contract rUSDYFactory is IMulticall {
     ExCallData[] calldata exCallData
   ) external payable override onlyGuardian returns (bytes[] memory results) {
     results = new bytes[](exCallData.length);
-    for (uint256 i = 0; i < exCallData.length; ++i) {
+    for (uint256 i = 0; i < exCallData.length; ) {
       (bool success, bytes memory ret) = address(exCallData[i].target).call{
         value: exCallData[i].value
       }(exCallData[i].data);
       require(success, "Call Failed");
       results[i] = ret;
+
+      unchecked {
+        ++i;
+      }
     }
   }
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L130
