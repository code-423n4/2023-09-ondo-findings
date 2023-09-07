## [G-01] use if + revert instead of require/assert to save gas
Consider using custom errors instead of revert strings. Can save gas when the revert condition has been met during runtime.
Here are some examples of where this optimization could be used:
```solidity
File: contracts/bridge/SourceBridge.sol
168:       require(success, "Call Failed");

File: contracts/rwaOracles/RWADynamicOracle.sol
405:     require(y == 0 || (z = x * y) / y == x);

File: contracts/usdy/rUSDYFactory.sol
100:     assert(rUSDYProxyAdmin.owner() == guardian);
134:       require(success, "Call Failed");
155:     require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");

File: contracts/usdy/rUSDY.sol
307:     require(currentAllowance >= _amount, "TRANSFER_AMOUNT_EXCEEDS_ALLOWANCE");
358:     require(
      currentAllowance >= _subtractedValue,
      "DECREASED_ALLOWANCE_BELOW_ZERO"
    );
435:     require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
450:     require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
490:     require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");
491:     require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");
519:     require(_sender != address(0), "TRANSFER_FROM_THE_ZERO_ADDRESS");
520:     require(_recipient != address(0), "TRANSFER_TO_THE_ZERO_ADDRESS");
525:     require(
      _sharesAmount <= currentSenderShares,
      "TRANSFER_AMOUNT_EXCEEDS_BALANCE"
    );
547:     require(_recipient != address(0), "MINT_TO_THE_ZERO_ADDRESS");
579:     require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");
584:     require(_sharesAmount <= accountShares, "BURN_AMOUNT_EXCEEDS_BALANCE");
634:       require(!_isBlocked(msg.sender), "rUSDY: 'sender' address blocked");
635:       require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");
636:       require(
        _isAllowed(msg.sender),
        "rUSDY: 'sender' address not on allowlist"
      );
644:       require(!_isBlocked(from), "rUSDY: 'from' address blocked");
645:       require(!_isSanctioned(from), "rUSDY: 'from' address sanctioned");
646:       require(_isAllowed(from), "rUSDY: 'from' address not on allowlist");
651:       require(!_isBlocked(to), "rUSDY: 'to' address blocked");
652:       require(!_isSanctioned(to), "rUSDY: 'to' address sanctioned");
653:       require(_isAllowed(to), "rUSDY: 'to' address not on allowlist");

```
 - SourceBridge.sol : [[168](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L168)]
 - RWADynamicOracle.sol : [[405](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L405)]
 - rUSDYFactory.sol : [[100](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L100), [134](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L134), [155](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L155)]
 - rUSDY.sol : [[307](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L307), [358](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L358), [435](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L435), [450](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L450), [490](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L490), [491](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L491), [519](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L519), [520](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L520), [525](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L525), [547](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L547), [579](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L579), [584](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L584), [634](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L634), [635](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L635), [636](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L636), [644](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L644), [645](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L645), [646](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L646), [651](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L651), [652](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L652), [653](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L653)]

Note that this will only decrease runtime gas when the revert condition has been met. Regardless, it will decrease deploy time gas


## [G-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops
```solidity
File: contracts/bridge/DestinationBridge.sol
134: for (uint256 i = 0; i < thresholds.length; ++i) {
160: for (uint256 i = 0; i < t.approvers.length; ++i) {
264: for (uint256 i = 0; i < amounts.length; ++i) {

File: contracts/bridge/SourceBridge.sol
79: bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
164: for (uint256 i = 0; i < exCallData.length; ++i) {

File: contracts/rwaOracles/RWADynamicOracle.sol
77: for (uint256 i = 0; i < length; ++i) {
113: for (uint256 i = 0; i < length; ++i) {
129: for (uint256 i = 0; i < length + 1; ++i) {

File: contracts/usdy/rUSDYFactory.sol
130: for (uint256 i = 0; i < exCallData.length; ++i) {

```
 - DestinationBridge.sol : [[134](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L134), [160](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L160), [264](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L264)]
 - SourceBridge.sol : [[79](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L79), [164](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L164)]
 - RWADynamicOracle.sol : [[77](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L77), [113](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L113), [129](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L129)]
 - rUSDYFactory.sol : [[130](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L130)]

## [G-03] <array>.length should not be looked up in every loop of a for-loop
The overheads outlined below are PER LOOP, excluding the first loop
- storage arrays incur a Gwarmaccess (**100 gas**)
- memory arrays use MLOAD (**3 gas**)
- calldata arrays use CALLDATALOAD (**3 gas**)
Caching the length changes each of these to a DUP<N> (**3 gas**), and gets rid of the extra DUP<N> needed to store the stack offset
Here are some examples of where this optimization could be used:
```solidity
File: contracts/bridge/DestinationBridge.sol
134: for (uint256 i = 0; i < thresholds.length; ++i) {
160: for (uint256 i = 0; i < t.approvers.length; ++i) {
264: for (uint256 i = 0; i < amounts.length; ++i) {

File: contracts/bridge/SourceBridge.sol
164: for (uint256 i = 0; i < exCallData.length; ++i) {

File: contracts/rwaOracles/RWADynamicOracle.sol
77: for (uint256 i = 0; i < length; ++i) {
113: for (uint256 i = 0; i < length; ++i) {
129: for (uint256 i = 0; i < length + 1; ++i) {

File: contracts/usdy/rUSDYFactory.sol
130: for (uint256 i = 0; i < exCallData.length; ++i) {

```
 - DestinationBridge.sol : [[134](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L134), [160](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L160), [264](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L264)]
 - SourceBridge.sol : [[164](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L164)]
 - RWADynamicOracle.sol : [[77](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L77), [113](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L113), [129](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L129)]
 - rUSDYFactory.sol : [[130](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L130)]

## [G-04] mark event arguments up to 3 arguments as indexed to save gas
Using this keyword with value types such as `uint`, `bool`, and `address` saves gas; however, this is only true for these value types, since indexing `bytes`, `strings` and `arrays` is more costly than not indexing them.

Here are some examples of where this optimization could be used:

```solidity
File: contracts/bridge/DestinationBridge.sol
389:   event ApproverRemoved(address approver);
396:   event ApproverAdded(address approver);
422:   event BridgeCompleted(address user, uint256 amount);
432:   event MessageReceived(
    string srcChain,
    address srcSender,
    uint256 amt,
    uint256 nonce
  );

File: contracts/bridge/SourceBridge.sol
183:   event DestinationChainContractAddressSet(
    string indexed destinationChain,
    address contractAddress
  );

File: contracts/rwaOracles/RWADynamicOracle.sol
309:   event RangeSet(
    uint256 indexed index,
    uint256 start,
    uint256 end,
    uint256 dailyInterestRate,
    uint256 prevRangeClosePrice
  );
326:   event RangeOverriden(
    uint256 indexed index,
    uint256 newStart,
    uint256 newEnd,
    uint256 newDailyInterestRate,
    uint256 newPrevRangeClosePrice
  );

File: contracts/usdy/rUSDYFactory.sol
146:   event rUSDYDeployed(
    address proxy,
    address proxyAdmin,
    address implementation,
    string name,
    string ticker
  );

File: contracts/usdy/rUSDY.sol
154:   event TransferShares(
    address indexed from,
    address indexed to,
    uint256 sharesValue
  );
172:   event SharesBurnt(
    address indexed account,
    uint256 preRebaseTokenAmount,
    uint256 postRebaseTokenAmount,
    uint256 sharesAmount
  );
189:   event TokensBurnt(address indexed account, uint256 tokensBurnt);

```
 - DestinationBridge.sol : [[389](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L389), [396](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L396), [422](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L422), [432](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L432)]
 - SourceBridge.sol : [[183](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L183)]
 - RWADynamicOracle.sol : [[309](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L309), [326](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L326)]
 - rUSDYFactory.sol : [[146](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L146)]
 - rUSDY.sol : [[154](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L154), [172](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L172), [189](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L189)]

## [G-05] <x> += <y> costs more gas than <x> = <x> + <y> for state variables
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

Here are some examples of where this optimization could be used:

```solidity
File: contracts/usdy/rUSDY.sol
551: totalShares += _sharesAmount;
588: totalShares -= _sharesAmount;

```
 - rUSDY.sol : [[551](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L551), [588](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L588)]

## [G-06] Usage of `int`s/`uint`s smaller than 32 bytes incurs overhead
Using ints/uints smaller than 32 bytes may cost more gas. This is because the EVM operates on 32 bytes at a time, so if an element is smaller than 32 bytes, the EVM must perform more operations to reduce the size of the element from 32 bytes to the desired size.

Here are some examples of where this optimization could be used:

```solidity
File: contracts/usdy/rUSDY.sol
209: function decimals() public pure returns (uint8) {

```
 - rUSDY.sol : [[209](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L209)]

## [G-07] Using a double if statement instead of a logical AND (&&) can provide similar short-circuiting behavior whereas double if is [slightly more gas efficient](https://gist.github.com/DadeKuma/931ce6794a050201ec6544dbcc31316c).

Here are some examples of where this optimization could be used:

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol
201: if (rangeLength > 1 && newEnd > ranges[indexToModify + 1].start)

File: contracts/usdy/rUSDY.sol
633: if (from != msg.sender && to != msg.sender) {

```
 - RWADynamicOracle.sol : [[201](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L201)]
 - rUSDY.sol : [[633](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L633)]

## [G-08] Using private for constants saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

Here are some examples of where this optimization could be used:

```solidity
File: contracts/bridge/DestinationBridge.sol
48: bytes32 public constant VERSION = "1.0";

File: contracts/bridge/SourceBridge.sol
27: bytes32 public constant VERSION = "1.0";

File: contracts/rwaOracles/RWADynamicOracle.sol
23: uint256 public constant DAY = 1 days;
27: bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
28: bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
343: uint256 private constant ONE = 10 complicated_reverts const_vars dumb_arithmetics events ifs inc_dec_unchecked lengths lengths_in_loops loops memory_function_args report.md requires small_types strings structs 27;

File: contracts/usdy/rUSDYFactory.sol
44: bytes32 public constant DEFAULT_ADMIN_ROLE = bytes32(0);

File: contracts/usdy/rUSDY.sol
91: uint256 public constant BPS_DENOMINATOR = 10_000;
97: bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
98: bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
99: bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
100: bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
101: bytes32 public constant LIST_CONFIGURER_ROLE =

```
 - DestinationBridge.sol : [[48](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L48)]
 - SourceBridge.sol : [[27](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L27)]
 - RWADynamicOracle.sol : [[23](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L23), [27](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L27), [28](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L28), [343](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L343)]
 - rUSDYFactory.sol : [[44](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L44)]
 - rUSDY.sol : [[91](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L91), [97](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L97), [98](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L98), [99](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L99), [100](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L100), [101](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L101)]