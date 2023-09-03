## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

```solidity
// place this struct with the state variables.
295:  struct Range {

// place these events before the constructor
309:  event RangeSet(
326:  event RangeOverriden(

// place these errors before the constructor
334:  error InvalidPrice();
335:  error InvalidRange();
336:  error PriceNotSet();

// place this state variable with the other ones
343:  uint256 private constant ONE = 10 ** 27;
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol

```solidity
// place this modifier before the constructor
154:  modifier onlyGuardian() {

// place this event before the constructor
146:  event rUSDYDeployed(
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol

```solidity
// place these events right after state variables.
154:  event TransferShares(
172:  event SharesBurnt(
189:  event TokensBurnt(address indexed account, uint256 tokensBurnt);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol

```solidity
/// place these before the constructor
183:  event DestinationChainContractAddressSet(
189:  error DestinationNotSupported();
190:  error GasFeeTooLow();
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

```solidity
// place these structs with the other state variables
369:  struct Threshold {
374:  struct TxnThreshold {
379:  struct Transaction {

// place these events before the constructor
389:  event ApproverRemoved(address approver);
396:  event ApproverAdded(address approver);
405:  event ChainIdSupported(string srcChain, string approvedSource);
414:  event ThresholdSet(string chain, uint256[] amounts, uint256[] numOfApprovers);
422:  event BridgeCompleted(address user, uint256 amount);
432:  event MessageReceived(

// place these errors before the constructor
439:  error NotApprover();
440:  error NoThresholdMatch();
441:  error ThresholdsNotInAscendingOrder();
443:  error ChainNotSupported();
444:  error SourceNotSupported();
445:  error NonceSpent();
446:  error AlreadyApproved();
447:  error InvalidVersion();
448:  error ArrayLengthMismatch();
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

```solidity
/// place these external functions before all the other visibility types
104:  function simulateRange(
151:  function setRange(
186:  function overrideRange(
241:  function pauseOracle() external onlyRole(PAUSER_ROLE) {
248:  function unpauseOracle() external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol

```solidity
// place internal functions for last since there are no private ones.
120:  function __rUSDY_init(
134:  function __rUSDY_init_unchained(
463:  function _transfer(
485:  function _approve(
500:  function _sharesOf(address _account) internal view returns (uint256) {
514:  function _transferShares(
543:  function _mintShares(
575:  function _burnShares(
626:  function _beforeTokenTransfer(


// place these external functions before all the other visibilities
434:  function wrap(uint256 _USDYAmount) external whenNotPaused {
449:  function unwrap(uint256 _rUSDYAmount) external whenNotPaused {
662:  function setOracle(address _oracle) external onlyRole(USDY_MANAGER_ROLE) {
672:  function burn(
685:  function pause() external onlyRole(PAUSER_ROLE) {
689:  function unpause() external onlyRole(USDY_MANAGER_ROLE) {
698:  function setBlocklist(
709:  function setAllowlist(
720:  function setSanctionsList(
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

```solidity
// place these external functions first, before all the internal ones.
197:  function approve(bytes32 txnHash) external {
210:  function addApprover(address approver) external onlyOwner {
220:  function removeApprover(address approver) external onlyOwner {
234:  function addChainSupport(
255:  function setThresholds(
286:  function setMintLimit(uint256 mintLimit) external onlyOwner {
295:  function setMintLimitDuration(uint256 mintDuration) external onlyOwner {
304:  function pause() external onlyOwner {
313:  function unpause() external onlyOwner {
322:  function rescueTokens(address _token) external onlyOwner {
361:  function getNumApproved(bytes32 txnHash) external view returns (uint256) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol

```solidity
// place this private function for last.
91:  function _payGasAndCallContract(
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/IRWADynamicOracle.sol

```solidity
18:  interface IRWADynamicOracle {

// @return missing
20:  function getPrice() external view returns (uint256);
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

```solidity
30:  constructor(
295:  struct Range {
334:  error InvalidPrice();
335:  error InvalidRange();
336:  error PriceNotSet();
345:  function _rpow(
400:  function _rmul(uint256 x, uint256 y) internal pure returns (uint256 z) {
404:  function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {

// @return missing
104:  function simulateRange(
262:  function derivePrice(
282:  function roundUpTo8(uint256 value) internal pure returns (uint256) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol

```solidity
51:  constructor(address _guardian) {

// @return missing
126:  function multiexcall(

154:  modifier onlyGuardian() {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol

```solidity
109:  function initialize(
120:  function __rUSDY_init(
134:  function __rUSDY_init_unchained(
685:  function pause() external onlyRole(PAUSER_ROLE) {
698:  function unpause() external onlyRole(USDY_MANAGER_ROLE) {


// @params missing
154:  event TransferShares(
388:  function getSharesByRUSDY(
397:  function getRUSDYByShares(uint256 _shares) public view returns (uint256) {
463:  function _transfer(
500:  function _sharesOf(address _account) internal view returns (uint256) {

// @return missing
543:  function _mintShares(
575:  function _burnShares(
626:  function _beforeTokenTransfer(

// dev/notice missing
194:  function name() public pure returns (string memory) {
202:  function symbol() public pure returns (string memory) {
209:  function decimals() public pure returns (uint8) {
216:  function totalSupply() public view returns (uint256) {
256:  function allowance(
372:  function getTotalShares() public view returns (uint256) {
381:  function sharesOf(address _account) public view returns (uint256) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

```solidity
27:  contract DestinationBridge is
55:  constructor(
369:  struct Threshold {
374:  struct TxnThreshold {
379:  struct Transaction {
439:  error NotApprover();
440:  error NoThresholdMatch();
441:  error ThresholdsNotInAscendingOrder();
443:  error ChainNotSupported();
444:  error SourceNotSupported();
445:  error NonceSpent();
446:  error AlreadyApproved();
447:  error InvalidVersion();
448:  error ArrayLengthMismatch();

/// @return missing
177:  function _checkThresholdMet(bytes32 txnHash) internal view returns (bool) {
361:  function getNumApproved(bytes32 txnHash) external view returns (uint256) {

// @dev/notice missing
197:  function approve(bytes32 txnHash) external {
210:  function addApprover(address approver) external onlyOwner {
220:  function removeApprover(address approver) external onlyOwner {
286:  function setMintLimit(uint256 mintLimit) external onlyOwner {
295:  function setMintLimitDuration(uint256 mintDuration) external onlyOwner {
337:  function _mintIfThresholdMet(bytes32 txnHash) internal {

```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol

```solidity
11:  contract SourceBridge is Ownable, Pausable, IMulticall {

/// @return missing
160:  function multiexcall(

189:  error DestinationNotSupported();
190:  error GasFeeTooLow();
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

```solidity
262:  function derivePrice(
282:  function roundUpTo8(uint256 value) internal pure returns (uint256) {
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol

```solidity
// private and internal `variables` should preppend with `underline`
46:  address internal immutable guardian;
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol

```solidity
//  private and internal `variables` should preppend with `underline`
76:  mapping(address => uint256) private shares;
79:  mapping(address => mapping(address => uint256)) private allowances;
82:  uint256 private totalShares;
```