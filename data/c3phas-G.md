# Codebase Optimization Report

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table of contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table of contents](#table-of-contents)
  - [Codebase impressions](#codebase-impressions)
  - [Note on Gas estimates.](#note-on-gas-estimates)
  - [Pack structs by putting data types that can fit together next to each other](#pack-structs-by-putting-data-types-that-can-fit-together-next-to-each-other)
    - [We can pack start and end together by reducing their size to `uint48` since they are just timestamps(save 1 SLOT: 2K Gas)](#we-can-pack-start-and-end-together-by-reducing-their-size-to-uint48-since-they-are-just-timestampssave-1-slot-2k-gas)
    - [The function should first verify if msg.value \> 0 before performing any other operation(Save 7146  Gas)](#the-function-should-first-verify-if-msgvalue--0-before-performing-any-other-operationsave-7146--gas)
  - [Optimize the function `_mintIfThresholdMet()`](#optimize-the-function-_mintifthresholdmet)
    - [Only read state if we are going to execute the entire logic(Saves ~2000 gas for the state read)](#only-read-state-if-we-are-going-to-execute-the-entire-logicsaves-2000-gas-for-the-state-read)
  - [Optimize check order: avoid making any state reads/writes if we have to validate some function parameters](#optimize-check-order-avoid-making-any-state-readswrites-if-we-have-to-validate-some-function-parameters)
  - [Don't cache calls that are only used once](#dont-cache-calls-that-are-only-used-once)
    - [DestinationBridge.sol.rescueTokens(): Save 8 gas](#destinationbridgesolrescuetokens-save-8-gas)
    - [DestinationBridge.sol.\_mintIfThresholdMet(): Save 10 gas](#destinationbridgesol_mintifthresholdmet-save-10-gas)
  - [Refactor the assembly code](#refactor-the-assembly-code)
  - [Conclusion](#conclusion)


## Codebase impressions

The developers have done an excellent job of identifying and implementing some of the most evident optimizations. 

However, we have identified additional areas where optimization is possible, some of which may not be immediately apparent.

## Note on Gas estimates.

We've tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average gas before and after will be included, and often a diff of the code will also accompany this.

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the opcodes involved. 


## Pack structs by putting data types that can fit together next to each other

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

https://github.com/code-423n4/2023-09-ondo/blob/02ebedeeb85b708345a4deb53a2b543ecae160d0/contracts/rwaOracles/RWADynamicOracle.sol#L295-L300

### We can pack start and end together by reducing their size to `uint48` since they are just timestamps(save 1 SLOT: 2K Gas)

```solidity
File: /contracts/rwaOracles/RWADynamicOracle.sol
295:  struct Range {
296:    uint256 start;
297:    uint256 end;
298:    uint256 dailyInterestRate;
299:    uint256 prevRangeClosePrice;
300:  }
```

```diff
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index 03aa7d6..6b8ff93 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -293,8 +293,8 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
   //////////////////////////////////////////////////////////////*/

   struct Range {
-    uint256 start;
-    uint256 end;
+    uint48 start;
+    uint48 end;
     uint256 dailyInterestRate;
     uint256 prevRangeClosePrice;
   }
```


https://github.com/code-423n4/2023-09-ondo/blob/02ebedeeb85b708345a4deb53a2b543ecae160d0/contracts/bridge/SourceBridge.sol#L61-L74

### The function should first verify if msg.value > 0 before performing any other operation(Save 7146  Gas)

**In case of a revert on `msg.value == 0` we save 7146 gas on average from the protocol gas tests(to simulate the sad path ie `msg.value == 0` see the test modification)**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 9815    | 9815   | 9815 | 9815 |
| After  | 2669    | 2669   | 2669 | 2669 |

```solidity
File: /contracts/bridge/SourceBridge.sol
61:  function burnAndCallAxelar(
62:    uint256 amount,
63:    string calldata destinationChain
64:  ) external payable whenNotPaused {
65:    // check destinationChain is correct
66:    string memory destContract = destChainToContractAddr[destinationChain];

68:    if (bytes(destContract).length == 0) {
69:      revert DestinationNotSupported();
70:    }

72:    if (msg.value == 0) {
73:      revert GasFeeTooLow();
74:    }
```


```diff
diff --git a/contracts/bridge/SourceBridge.sol b/contracts/bridge/SourceBridge.sol
index 457bf41..b436ff7 100644
--- a/contracts/bridge/SourceBridge.sol
+++ b/contracts/bridge/SourceBridge.sol
@@ -62,6 +62,10 @@ contract SourceBridge is Ownable, Pausable, IMulticall {
     uint256 amount,
     string calldata destinationChain
   ) external payable whenNotPaused {
+
+    if (msg.value == 0) {
+      revert GasFeeTooLow();
+    }
     // check destinationChain is correct
     string memory destContract = destChainToContractAddr[destinationChain];

@@ -69,10 +73,6 @@ contract SourceBridge is Ownable, Pausable, IMulticall {
       revert DestinationNotSupported();
     }

-    if (msg.value == 0) {
-      revert GasFeeTooLow();
-    }
```


To get the perfect representation of the gas being saved, I modified the test function test_bridge() as follows

```diff
   function test_bridge() public initializeAlice {
     uint256 gasReceiverBalanceBefore = AXELAR_GAS_SERVICE.balance;

     // Expect event emit from gas receiver
     bytes memory payload = abi.encode(bytes32("1.0"), alice, 100e18, 0);
     bytes32 payloadHash = keccak256(payload);
-    vm.expectEmit(true, true, true, true);
+    //vm.expectEmit(true, true, true, true);
     emit NativeGasPaidForContractCall(
       address(usdyBridge),
       "optimism",
@@ -160,7 +160,7 @@ contract Test_SourceBridge is USDY_BasicDeployment, SourceBridgeEvents {
     );

     // Expect event emit from gateway
-    vm.expectEmit(true, true, true, true);
+    //vm.expectEmit(true, true, true, true);
     emit ContractCall(
       address(usdyBridge),
       "optimism",
@@ -171,7 +171,7 @@ contract Test_SourceBridge is USDY_BasicDeployment, SourceBridgeEvents {

     // Bridge Tokens
     vm.prank(alice);
-    usdyBridge.burnAndCallAxelar{value: 0.01 ether}(100e18, "optimism");
+    usdyBridge.burnAndCallAxelar{value: 0.00 ether}(100e18, "optimism");

     assertEq(usdy.balanceOf(address(usdyBridge)), 0);
     assertEq(usdy.balanceOf(alice), 0);
```

Note, using the `vm.expectRevert` also produces the same benchmarks 


## Optimize the function `_mintIfThresholdMet()` 

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L337-L353

### Only read state if we are going to execute the entire logic(Saves ~2000 gas for the state read)

```solidity
File: /contracts/bridge/DestinationBridge.sol
337:  function _mintIfThresholdMet(bytes32 txnHash) internal {
338:    bool thresholdMet = _checkThresholdMet(txnHash);
339:    Transaction memory txn = txnHashToTransaction[txnHash];
340:    if (thresholdMet) {
341:      _checkAndUpdateInstantMintLimit(txn.amount);
342:      if (!ALLOWLIST.isAllowed(txn.sender)) {
343:        ALLOWLIST.setAccountStatus(
344:          txn.sender,
345:          ALLOWLIST.getValidTermIndexes()[0],
346:          true
347:        );
348:      }
349:      TOKEN.mint(txn.sender, txn.amount);
350:      delete txnHashToTransaction[txnHash];
351:      emit BridgeCompleted(txn.sender, txn.amount);
352:    }
353:  }
```


The function above, only mints if threshold is met. Before checking the status of whether the threshold is met or not, we making a state read(`Transaction memory txn = txnHashToTransaction[txnHash];`) which is quite expensive. we then proceed to check if threshold is met and execute the function if threshold is indeed met.
If not met, we would not execute anything but we would still have made the state read, as it's being read before the check

We can refactor the function to have the check before to avoid wasting gas in case threshold is not met. 

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index 8ad410c..13cc444 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -336,8 +336,8 @@ contract DestinationBridge is
    */
   function _mintIfThresholdMet(bytes32 txnHash) internal {
     bool thresholdMet = _checkThresholdMet(txnHash);
-    Transaction memory txn = txnHashToTransaction[txnHash];
     if (thresholdMet) {
+      Transaction memory txn = txnHashToTransaction[txnHash];
       _checkAndUpdateInstantMintLimit(txn.amount);
       if (!ALLOWLIST.isAllowed(txn.sender)) {
         ALLOWLIST.setAccountStatus(
```


## Optimize check order: avoid making any state reads/writes if we have to validate some function parameters

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L30-L46
```solidity
File: /contracts/rwaOracles/RWADynamicOracle.sol
30:  constructor(
31:    address admin,
32:    address setter,
33:    address pauser,
34:    uint256 firstRangeStart,
35:    uint256 firstRangeEnd,
36:    uint256 dailyIR,
37:    uint256 startPrice
38:  ) {
39:    _grantRole(DEFAULT_ADMIN_ROLE, admin);
40:    _grantRole(PAUSER_ROLE, pauser);
41:    _grantRole(SETTER_ROLE, setter);

43:    if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
44:    uint256 trueStart = (startPrice * ONE) / dailyIR;
45:    ranges.push(Range(firstRangeStart, firstRangeEnd, dailyIR, trueStart));
46:  }
```

We have a check for function parameters which reverts the whole transaction if `firstRangeStart >= firstRangeEnd` In case of a revert, all the gas consumed so far will not be refunded. If we look at our constructor the operations that happen before this check, consume so much gas as they involve reading and writing states.
The `_grantRole()` is inherited and the function has the following implementation
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/external/openzeppelin/contracts/access/AccessControl.sol#L238-L243
```solidity
File: /contracts/external/openzeppelin/contracts/access/AccessControl.sol
238:  function _grantRole(bytes32 role, address account) internal virtual {
239:    if (!hasRole(role, account)) {
240:      _roles[role].members[account] = true;
241:      emit RoleGranted(role, account, _msgSender());
242:    }
243:  }
```
Now the above function is called 3 times in our constructor. Most of this reads/writes will be cold thus the cost would be higher.

```diff
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index 03aa7d6..fd07948 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -36,11 +36,11 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
     uint256 dailyIR,
     uint256 startPrice
   ) {
+        if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
     _grantRole(DEFAULT_ADMIN_ROLE, admin);
     _grantRole(PAUSER_ROLE, pauser);
     _grantRole(SETTER_ROLE, setter);

-    if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
     uint256 trueStart = (startPrice * ONE) / dailyIR;
     ranges.push(Range(firstRangeStart, firstRangeEnd, dailyIR, trueStart));
   }
```

## Don't cache calls that are only used once

https://github.com/code-423n4/2023-09-ondo/blob/02ebedeeb85b708345a4deb53a2b543ecae160d0/contracts/bridge/DestinationBridge.sol#L322-L325

### DestinationBridge.sol.rescueTokens(): Save 8 gas

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2608    | 16302   | 16302 | 29996 |
| After  | 2608    | 16294   | 16294 | 29980 |
```solidity
File: /contracts/bridge/DestinationBridge.sol
322:  function rescueTokens(address _token) external onlyOwner {
323:    uint256 balance = IRWALike(_token).balanceOf(address(this));
324:    IRWALike(_token).transfer(owner(), balance);
325:  }
```

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index 8ad410c..376f1c3 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -320,8 +320,7 @@ contract DestinationBridge is
    * @param _token The address of the token to rescue
    */
   function rescueTokens(address _token) external onlyOwner {
-    uint256 balance = IRWALike(_token).balanceOf(address(this));
-    IRWALike(_token).transfer(owner(), balance);
+    IRWALike(_token).transfer(owner(), IRWALike(_token).balanceOf(address(this)));
   }
```

https://github.com/code-423n4/2023-09-ondo/blob/02ebedeeb85b708345a4deb53a2b543ecae160d0/contracts/bridge/DestinationBridge.sol#L337-L353

### DestinationBridge.sol.\_mintIfThresholdMet(): Save 10 gas

**Benchmarks based on function `approve()` which calls our internal function

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 1801    | 62199   | 27240 | 159159 |
| After  | 1801    | 62189   | 27224 | 159146 |
```solidity
File: /contracts/bridge/DestinationBridge.sol
337:  function _mintIfThresholdMet(bytes32 txnHash) internal {
338:    bool thresholdMet = _checkThresholdMet(txnHash);
339:    Transaction memory txn = txnHashToTransaction[txnHash];
340:    if (thresholdMet) {
341:      _checkAndUpdateInstantMintLimit(txn.amount);
```

```diff
diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
index 8ad410c..4562f20 100644
--- a/contracts/bridge/DestinationBridge.sol
+++ b/contracts/bridge/DestinationBridge.sol
@@ -335,9 +335,8 @@ contract DestinationBridge is
    * @param txnHash The hash of the transaction we wish to mint
    */
   function _mintIfThresholdMet(bytes32 txnHash) internal {
-    bool thresholdMet = _checkThresholdMet(txnHash);
     Transaction memory txn = txnHashToTransaction[txnHash];
-    if (thresholdMet) {
+    if (_checkThresholdMet(txnHash)) {
       _checkAndUpdateInstantMintLimit(txn.amount);
       if (!ALLOWLIST.isAllowed(txn.sender)) {
         ALLOWLIST.setAccountStatus(
```

## Refactor the assembly code 

https://github.com/code-423n4/2023-09-ondo/blob/02ebedeeb85b708345a4deb53a2b543ecae160d0/contracts/rwaOracles/RWADynamicOracle.sol#L369-L374
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4021    | 7231   | 7834 | 17031 |
| After  | 4017    | 7217   | 7822 | 17015 |

The `div` opcode uses 5 gas while the `shr` opcode uses 3 gas. Since the denominator is known(2) we can simply shift right by 1 which is equivalent to division by 2
```solidity
File: /contracts/rwaOracles/RWADynamicOracle.sol
369:        let half := div(base, 2) // for rounding.
370:        for {
371:          n := div(n, 2)
372:        } n {
373:          n := div(n, 2)
374:        } {
```

```diff
diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
index 03aa7d6..fa175e0 100644
--- a/contracts/rwaOracles/RWADynamicOracle.sol
+++ b/contracts/rwaOracles/RWADynamicOracle.sol
@@ -366,11 +366,11 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
         default {
           z := x
         }
-        let half := div(base, 2) // for rounding.
+        let half := shr(1, base) // for rounding.
         for {
-          n := div(n, 2)
+          n := shr(1, n)
         } n {
-          n := div(n, 2)
+          n := shr(1, n)
         } {
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
