| Number | Optimization Details                                                               | Instances |
| :----: | :--------------------------------------------------------------------------------- | :-------: |
| [G-01] | Do not calculate constant variables                                                                       |     7     |
| [G-02] | Non efficient zero initialization                                                                   |     7     |
| [G-03] | Using ternary operator instead of if else saves gas                                |     2     |
| [G-04] | Pre-increment and pre-decrement are cheaper as compared to post increment and post decrement                                       |     1     |

Total 4 issues

## [G-01] Do not calculate constant variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas each time of use.

_Total 7 instances - 1 files:_

### Instance#1-5 : Assign direct simple constant value after calculating off chain

```solidity
File : contracts/usdy/rUSDY.sol

97:  bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
98:  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
99:  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
100: bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
101: bytes32 public constant LIST_CONFIGURER_ROLE =
102:    keccak256("LIST_CONFIGURER_ROLE");

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97C3-L102C39

### Instance#6-7 : Assign direct simple constant value after calculating off chain

```solidity
File : contracts/rwaOracles/RWADynamicOracle.sol

27:  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
28:  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L27C3-L28C66

## [G-02] Non efficient zero initialization

_7 instances - 4 file:_

### Instance#1: Initialisation of i as zero is redundant

```solidity
File: contracts/bridge/SourceBridge.sol
164:    for (uint256 i = 0; i < exCallData.length; ++i) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L164

### Instance#2-4: Initialisation of i as zero is redundant

```solidity
File: contracts/bridge/DestinationBridge.sol
134:    for (uint256 i = 0; i < thresholds.length; ++i) {

160:    for (uint256 i = 0; i < t.approvers.length; ++i) {

264:    for (uint256 i = 0; i < amounts.length; ++i) {


```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L134
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L160
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L264

### Instance#5: Initialisation of i as zero is redundant

```solidity
File: contracts/usdy/rUSDYFactory.sol
130:     for (uint256 i = 0; i < exCallData.length; ++i) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L130

### Instance#6-7: Initialisation of i as zero is redundant

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol
113:for (uint256 i = 0; i < length; ++i) {

129:for (uint256 i = 0; i < length + 1; ++i) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L113
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L129s

## [G-03] Using ternary operator instead of if-else saves gas

_Total 2 instances - 2 file:_

### Instance#1:

```solidity
File: contracts/bridge/DestinationBridge.sol
179:if (t.numberOfApprovalsNeeded <= t.approvers.length) {
180:      return true;
181:    } else {
182:      return false;
183:    }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L179C5-L183C6

### Instance#2:

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol
132:if (range.end <= blockTimeStamp) {
133:          return derivePrice(range, range.end - 1);
134:        } else {
135:          return derivePrice(range, blockTimeStamp);
136:        }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L132C9-L136C10

## [G-04] Pre-increment and pre-decrement are cheaper as compared to post increment and post decrement

++i costs less gas compared to i++ or i += 1 for an unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled..

_Total 1 instances - 1 file:_

### Instance#1:

```solidity
File: contracts/bridge/SourceBridge.sol
//@audit  nonce++
79: bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79

