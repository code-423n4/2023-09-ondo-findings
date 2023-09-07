## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [[G-01](#g-01-structs-can-be-packed-into-fewer-storage-slots)] | `Structs` can be packed into fewer storage slots. | 1 | 
| [[G-02](#g-02-abiencode-is-less-efficient-than-abiencodepacked-to-save-gas)] | `abi.encode()` is less efficient than `abi.encodePacked()` to save gas. | 3 | 
| [[G-03](#g-03-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)] | Use `calldata` instead of `memory` for function arguments that do not get mutated. | 3 |
| [[G-04](#g-04-no-need-to-explicitly-initialize-variables-with-default-values)] | No need to explicitly initialize variables with `default` values. | 8 |
| [[G-05](#g-05-using-ternary-operator-instead-of-if-else-saves-gas)] | Using `ternary` operator instead of `if-else` saves gas | 1 |
| [[G-06](#g-06-use-hardcode-address-instead-addressthis)] | Use `hardcode address` instead `address(this)` | 2 |
| [[G-07](#g-07-change-constant-to-immutable-for-keccak-variables-20-gas-per-keccak)] | Change `Constant` to `Immutable` for keccak Variables (20 gas per keccak) | 7 |

Total 7 issues.

## [G-01] Structs can be packed into fewer storage slots.

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

**_1 Instance - 1 File_**

### Reduce uint type for `start`,`end` and `dailyInterestRate` and pack them into 1 storage slot to save 2 SLOT (~4000 Gas) 

These  `start`,`end`  can be reduced to `uint64` because they are holding timestamps and `uint64` is more than enough to hold timestamps.

`dailyInterestRate`  are holding *%* value with **1e27** precision. And max  value a `uint128` can hold is **~ 3.40 × 10^38** . So `uint128` is big enough to hold `dailyInterestRate`.

```solidity
File : contracts/rwaOracles/RWADynamicOracle.sol

295:   struct Range {
296:    uint256 start;
297:    uint256 end;
298:    uint256 dailyInterestRate;
299:    uint256 prevRangeClosePrice;
300:  }
```
[295-300](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L295C1-L300C4)

```diff
File : contracts/rwaOracles/RWADynamicOracle.sol

  struct Range {
-   uint256 start;
-   uint256 end;
-   uint256 dailyInterestRate;
+   uint64 start;
+   uint64 end;
+   uint128 dailyInterestRate;
    uint256 prevRangeClosePrice;
  }

```

## [G-02] abi.encode() is less efficient than abi.encodePacked() to save gas.

In Solidity, abi.encode() and abi.encodePacked() are used to encode function arguments and data into a byte array for storage or transmission. However, abi.encodePacked() is generally more gas-efficient than abi.encode() because it does not add padding to the encoded data.
When using abi.encode(), the function arguments are encoded with padding to ensure that the encoded data is a multiple of 32 bytes. This padding can add unnecessary zeros to the encoded data, which can increase the size and gas cost of the transaction.
On the other hand, abi.encodePacked() does not add padding to the encoded data and returns a tightly packed representation of the input data. This can reduce the size and gas cost of the transaction, as it avoids unnecessary padding.

**_3 Instances - 2 Files_**

```diff
File : contracts/bridge/SourceBridge.sol

  function burnAndCallAxelar
  ...
- 79: bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

+ 79: bytes memory payload = abi.encodePacked(VERSION, msg.sender, amount, nonce++);

```
[79](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79)

```diff
File : contracts/bridge/DestinationBridge.sol

  function _execute
   ...
- 99: if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {
+ 99: if (chainToApprovedSender[srcChain] != keccak256(abi.encodePacked(srcAddr))) {


  function addChainSupport
   ...
- 238: chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));
+ 238: chainToApprovedSender[srcChain] = keccak256(abi.encodePacked(srcContractAddress));

```
[99](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L99),
[238](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L238)


## [G-03] Use calldata instead of memory for function arguments that do not get mutated.

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

**Note: These instances missed by bot report**

**_3 Instances - 2 Files_**

```solidity
File : contracts/bridge/SourceBridge.sol

91:  function _payGasAndCallContract(
92:    string calldata destinationChain,
93:    string memory destContract,
94:    bytes memory payload
95:  ) private {

```
[91-95](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L91C2-L95C14)

```diff
File : contracts/bridge/SourceBridge.sol

 function _payGasAndCallContract(
    string calldata destinationChain,
-   string memory destContract,
-   bytes memory payload
+   string calldata destContract,
+   bytes calldata payload
  ) private {
```

```solidity
File : contracts/bridge/DestinationBridge.sol

128: function _attachThreshold(
129:    uint256 amount,
130:    bytes32 txnHash,
131:    string memory srcChain
132:  ) internal {
```
[128-132](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L128C3-L132C15)

```diff
File : contracts/bridge/DestinationBridge.sol

 function _attachThreshold(
    uint256 amount,
    bytes32 txnHash,
-   string memory srcChain
+   string calldata srcChain
  ) internal {

```

## [G-04] No need to explicitly initialize variables with default values.

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address, etc.). Explicitly initializing it with its default value is an anti-pattern and wastes gas. 

**_8 Instances - 4 Files_**

### *`for (uint256 i = 0; i < numIterations; ++i)`*  should be replaced with: *`for (uint256 i; i < numIterations; ++i)`* 

```solidity
File : contracts/bridge/SourceBridge.sol

164: for (uint256 i = 0; i < exCallData.length; ++i) {

```
[164](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L164)

```solidity
Fle : contracts/bridge/DestinationBridge.sol

134: for (uint256 i = 0; i < thresholds.length; ++i) {

160: for (uint256 i = 0; i < t.approvers.length; ++i) {

264: for (uint256 i = 0; i < amounts.length; ++i) {

```
[134](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L134),
[160](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L160),
[264](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L264)

```solidity
File : contracts/rwaOracles/RWADynamicOracle.sol

 77: for (uint256 i = 0; i < length; ++i) {

113: for (uint256 i = 0; i < length; ++i) {    

129: for (uint256 i = 0; i < length + 1; ++i) {
```
[77](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L77),
[113](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L113)
[129](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L129)

```solidity
File : contracts/usdy/rUSDYFactory.sol

130: for (uint256 i = 0; i < exCallData.length; ++i) {
```
[130](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L130)


## [G-05] Using ternary operator instead of if else saves gas

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes conditional stack manipulation.
The exact amount of gas saved by using the ternary operator instead of the if-else construct in Solidity can vary depending on the specific scenario, the complexity of the surrounding code, and the conditions involved

**_1 Instance - 1 File_**

```solidity
File : contracts/bridge/DestinationBridge.sol

177: function _checkThresholdMet(bytes32 txnHash) internal view returns (bool) {
178:    TxnThreshold memory t = txnToThresholdSet[txnHash];
179:    if (t.numberOfApprovalsNeeded <= t.approvers.length) {
180:      return true;
181:    } else {
182:      return false;
183:    }
184:  }

```
[177-184](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L177C3-L184C4)

```diff
File : contracts/bridge/DestinationBridge.sol

 function _checkThresholdMet(bytes32 txnHash) internal view returns (bool) {
    TxnThreshold memory t = txnToThresholdSet[txnHash];
-   if (t.numberOfApprovalsNeeded <= t.approvers.length) {
-     return true;
-   } else {
-     return false;
-   }
+ return t.numberOfApprovalsNeeded <= t.approvers.length ? true : false;

  }
```

## [G-06] Use hardcode address instead address(this).

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. In Solidity, the address(this) expression returns the address of the current contract instance. This expression is commonly used to send or receive Ether to or from the contract.
Using a hardcoded address instead of the address(this) expression can be more gas-efficient when sending or receiving Ether to or from the contract. This is because using the address(this) expression requires additional gas to be consumed to retrieve the address of the current contract, while using a hardcoded address does not.

**_2 Instances - 2 Files_**

```solidity
File : contracts/bridge/DestinationBridge.sol

322: function rescueTokens(address _token) external onlyOwner {
323:    uint256 balance = IRWALike(_token).balanceOf(address(this));
324:    IRWALike(_token).transfer(owner(), balance);
325:  }

```
[322-325](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L322C2-L325C4)

```solidity
File : contracts/usdy/rUSDY.sol

434: function wrap(uint256 _USDYAmount) external whenNotPaused {
435:    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
436:    _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
437:    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
```
[434-437](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L434C3-L437C63)


## [G-07] Change Constant to Immutable for keccak Variables (20 gas per keccak)

The use of constant keccak variables results in extra hashing (and so gas). This results in the keccak operation being performed whenever the variable is used, increasing gas costs relative to just storing the output hash. Changing to immutable will only perform hashing on contract deployment which will save gas. You should use immutables until the referenced issues are implemented, then you only pay the gas costs for the computation at deploy time and you can save about `20 gas per one keccak`.

**_7 Instances - 2 Files_**

```diff
File : contracts/rwaOracles/RWADynamicOracle.sol

- 27:  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
- 28:  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

+ 27:  bytes32 public immutable SETTER_ROLE = keccak256("SETTER_ROLE");
+ 28:  bytes32 public immutable PAUSER_ROLE = keccak256("PAUSER_ROLE");

```
[27-28](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L27C3-L28C66)

```diff
File : contracts/usdy/rUSDY.sol

- 97:  bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
- 98:  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
- 99:  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
- 100: bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
- 101: bytes32 public constant LIST_CONFIGURER_ROLE =
  102:   keccak256("LIST_CONFIGURER_ROLE");

+ 97:  bytes32 public immutable USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
+ 98:  bytes32 public immutable MINTER_ROLE = keccak256("MINTER_ROLE");
+ 99:  bytes32 public immutable PAUSER_ROLE = keccak256("PAUSER_ROLE");
+ 100: bytes32 public immutable BURNER_ROLE = keccak256("BURN_ROLE");
+ 101: bytes32 public immutable LIST_CONFIGURER_ROLE =
  102:   keccak256("LIST_CONFIGURER_ROLE");

```
[97-102](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97C1-L102C39)