## Gas Optimizations

|        | Issue                                                                                                                  |
| ------ | :--------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                             |
| GAS-2  | Use assembly to check for `address(0)`                                                                                 |
| GAS-3  | Setting the constructor to payable                                                                                     |
| GAS-4  | DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION                                    |
| GAS-5  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                      |
| GAS-6  | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT                               |
| GAS-7  | KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE                                            |
| GAS-8  | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT                                                      |
| GAS-9  | Functions guaranteed to revert when called by normal users can be marked payable                                       |
| GAS-10 | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops |
| GAS-11 | The increment in for loop postcondition can be made unchecked                                                          |
| GAS-12 | Using `private` rather than `public` for constants, saves gas                                                          |
| GAS-13 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                    |
| GAS-14 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead                                                 |
| GAS-15 | USE BYTES32 INSTEAD OF STRING                                                                                          |

### [GAS-1] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

99:     if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {

238:     chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));

```

```solidity
File: contracts/bridge/SourceBridge.sol

79:     bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

```

### [GAS-2] Use assembly to check for `address(0)`

#### Description:

_Saves 6 gas per instance_

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

113:     emit MessageReceived(srcChain, srcSender, amt, nonce);

```

```solidity
File: contracts/usdy/rUSDY.sol

511:    * - `_sender` must hold at least `_sharesAmount` shares.

512:    * - the contract must not be paused.

536:    * @dev This doesn't increase the token total supply.

540:    * - `_recipient` cannot be the zero address.

560:     // as the result. This is equivalent to performing a send from each other token holder's

601:     return totalShares;

659:    * @dev The new oracle must comply with the `IPricerReader` interface

667:    * @notice Admin burn function to burn rUSDY tokens from any account

```

### [GAS-3] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

55:   constructor(

```

```solidity
File: contracts/bridge/SourceBridge.sol

40:   constructor(

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

30:   constructor(

```

```solidity
File: contracts/usdy/rUSDY.sol

105:   constructor() {

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

51:   constructor(address _guardian) {

```

### [GAS-4] DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION

#### **Proof Of Concept**

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

377:             revert(0, 0)

381:             revert(0, 0)

387:               revert(0, 0)

391:               revert(0, 0)

```

```solidity
File: contracts/usdy/rUSDY.sol

634:       require(!_isBlocked(msg.sender), "rUSDY: 'sender' address blocked");

635:       require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");

644:       require(!_isBlocked(from), "rUSDY: 'from' address blocked");

645:       require(!_isSanctioned(from), "rUSDY: 'from' address sanctioned");

646:       require(_isAllowed(from), "rUSDY: 'from' address not on allowlist");

651:       require(!_isBlocked(to), "rUSDY: 'to' address blocked");

652:       require(!_isSanctioned(to), "rUSDY: 'to' address sanctioned");

653:       require(_isAllowed(to), "rUSDY: 'to' address not on allowlist");

```

### [GAS-5] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDYFactory.sol

154:   modifier onlyGuardian() {

```

### [GAS-6] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### **Proof Of Concept**

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

27:   bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");

28:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

```solidity
File: contracts/usdy/rUSDY.sol

97:   bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");

98:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

99:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

100:   bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");

```

### [GAS-7] KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE

#### Description:

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

#### **Proof Of Concept**

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

28:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

```solidity
File: contracts/usdy/rUSDY.sol

99:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

### [GAS-8] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

48:   bytes32 public constant VERSION = "1.0";

```

```solidity
File: contracts/bridge/SourceBridge.sol

27:   bytes32 public constant VERSION = "1.0";

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

23:   uint256 public constant DAY = 1 days;

27:   bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");

28:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

```solidity
File: contracts/usdy/rUSDY.sol

91:   uint256 public constant BPS_DENOMINATOR = 10_000;

97:   bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");

98:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

99:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

100:   bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");

101:   bytes32 public constant LIST_CONFIGURER_ROLE =

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

44:   bytes32 public constant DEFAULT_ADMIN_ROLE = bytes32(0);

```

### [GAS-9] Functions guaranteed to revert when called by normal users can be marked payable

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

210:   function addApprover(address approver) external onlyOwner {

220:   function removeApprover(address approver) external onlyOwner {

237:   ) external onlyOwner {

259:   ) external onlyOwner {

286:   function setMintLimit(uint256 mintLimit) external onlyOwner {

295:   function setMintLimitDuration(uint256 mintDuration) external onlyOwner {

304:   function pause() external onlyOwner {

313:   function unpause() external onlyOwner {

322:   function rescueTokens(address _token) external onlyOwner {

```

```solidity
File: contracts/bridge/SourceBridge.sol

124:   ) external onlyOwner {

136:   function pause() external onlyOwner {

145:   function unpause() external onlyOwner {

162:   ) external payable override onlyOwner returns (bytes[] memory results) {

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

154:   ) external onlyRole(SETTER_ROLE) {

192:   ) external onlyRole(DEFAULT_ADMIN_ROLE) {

241:   function pauseOracle() external onlyRole(PAUSER_ROLE) {

248:   function unpauseOracle() external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: contracts/usdy/rUSDY.sol

127:   ) internal onlyInitializing {

138:   ) internal onlyInitializing {

662:   function setOracle(address _oracle) external onlyRole(USDY_MANAGER_ROLE) {

675:   ) external onlyRole(BURNER_ROLE) {

685:   function pause() external onlyRole(PAUSER_ROLE) {

689:   function unpause() external onlyRole(USDY_MANAGER_ROLE) {

700:   ) external override onlyRole(LIST_CONFIGURER_ROLE) {

711:   ) external override onlyRole(LIST_CONFIGURER_ROLE) {

722:   ) external override onlyRole(LIST_CONFIGURER_ROLE) {

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

81:   ) external onlyGuardian returns (address, address, address) {

128:   ) external payable override onlyGuardian returns (bytes[] memory results) {

154:   modifier onlyGuardian() {

```

### [GAS-10] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops

#### **Proof Of Concept**

```solidity
File: contracts/bridge/SourceBridge.sol

79:     bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

```

### [GAS-11] The increment in for loop postcondition can be made unchecked

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.
the for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: contracts/bridge/SourceBridge.sol

79:     bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

```

### [GAS-12] Using `private` rather than `public` for constants, saves gas

#### Description:

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

48:   bytes32 public constant VERSION = "1.0";

```

```solidity
File: contracts/bridge/SourceBridge.sol

27:   bytes32 public constant VERSION = "1.0";

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

23:   uint256 public constant DAY = 1 days;

27:   bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");

28:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

```solidity
File: contracts/usdy/rUSDY.sol

91:   uint256 public constant BPS_DENOMINATOR = 10_000;

97:   bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");

98:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

99:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

100:   bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");

101:   bytes32 public constant LIST_CONFIGURER_ROLE =

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

44:   bytes32 public constant DEFAULT_ADMIN_ROLE = bytes32(0);

```

### [GAS-13] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

179:  if (t.numberOfApprovalsNeeded <= t.approvers.length) {
      return true;
    } else {
      return false;
    }

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

80:     if (range.end <= block.timestamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, block.timestamp);
        }

132:    if (range.end <= blockTimeStamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, blockTimeStamp);
        }

```

### [GAS-14] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDY.sol

209:   function decimals() public pure returns (uint8) {

```

### [GAS-15] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

44:   mapping(string => bytes32) public chainToApprovedSender;

52:   mapping(string => Threshold[]) public chainToThresholds;

86:     string calldata srcChain,

87:     string calldata srcAddr,

131:     string memory srcChain

235:     string calldata srcChain,

236:     string calldata srcContractAddress

256:     string calldata srcChain,

405:   event ChainIdSupported(string srcChain, string approvedSource);

405:   event ChainIdSupported(string srcChain, string approvedSource);

414:   event ThresholdSet(string chain, uint256[] amounts, uint256[] numOfApprovers);

433:     string srcChain,

```

```solidity
File: contracts/bridge/SourceBridge.sol

15:   mapping(string => string) public destChainToContractAddr;

15:   mapping(string => string) public destChainToContractAddr;

63:     string calldata destinationChain

66:     string memory destContract = destChainToContractAddr[destinationChain];

92:     string calldata destinationChain,

93:     string memory destContract,

122:     string memory destinationChain,

184:     string indexed destinationChain,

```

```solidity
File: contracts/usdy/rUSDY.sol

194:   function name() public pure returns (string memory) {

202:   function symbol() public pure returns (string memory) {

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

150:     string name,

151:     string ticker

```
