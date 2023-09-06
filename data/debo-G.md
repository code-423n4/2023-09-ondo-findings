## [G-01] Do not initialize variables with default value
Description
Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::134 => for (uint256 i = 0; i < thresholds.length; ++i) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::160 => for (uint256 i = 0; i < t.approvers.length; ++i) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::264 => for (uint256 i = 0; i < amounts.length; ++i) {
2023-09-ondo/contracts/bridge/SourceBridge.sol::166 => for (uint256 i = 0; i < exCallData.length; ++i) {
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::77 => for (uint256 i = 0; i < length; ++i) {
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::113 => for (uint256 i = 0; i < length; ++i) {
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::129 => for (uint256 i = 0; i < length + 1; ++i) {
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::130 => for (uint256 i = 0; i < exCallData.length; ++i) {
```
## [G-02] Cache array length outside of loop
Description
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::134 => for (uint256 i = 0; i < thresholds.length; ++i) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::159 => if (t.approvers.length > 0) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::160 => for (uint256 i = 0; i < t.approvers.length; ++i) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::179 => if (t.numberOfApprovalsNeeded <= t.approvers.length) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::260 => if (amounts.length != numOfApprovers.length) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::264 => for (uint256 i = 0; i < amounts.length; ++i) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::362 => return txnToThresholdSet[txnHash].approvers.length;
2023-09-ondo/contracts/bridge/SourceBridge.sol::70 => if (bytes(destContract).length == 0) {
2023-09-ondo/contracts/bridge/SourceBridge.sol::165 => results = new bytes[](exCallData.length);
2023-09-ondo/contracts/bridge/SourceBridge.sol::166 => for (uint256 i = 0; i < exCallData.length; ++i) {
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::76 => uint256 length = ranges.length;
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::77 => for (uint256 i = 0; i < length; ++i) {
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::78 => Range storage range = ranges[(length - 1) - i];
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::111 => uint256 length = ranges.length;
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::112 => Range[] memory rangeList = new Range[](length + 1);
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::113 => for (uint256 i = 0; i < length; ++i) {
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::118 => rangeList[length] = Range(startTime, endTime, dailyIR, trueStart);
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::120 => Range memory lastRange = ranges[ranges.length - 1];
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::122 => rangeList[length] = Range(
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::129 => for (uint256 i = 0; i < length + 1; ++i) {
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::130 => Range memory range = rangeList[(length) - i];
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::155 => Range memory lastRange = ranges[ranges.length - 1];
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::165 => ranges.length - 1,
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::196 => uint256 rangeLength = ranges.length;
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::199 => // If the length of ranges is greater than 1,
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::129 => results = new bytes[](exCallData.length);
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::130 => for (uint256 i = 0; i < exCallData.length; ++i) {
2023-09-ondo/forge-tests/bridges/DestinationBridge.t.sol::436 => function test_Receiver_Owner_cannot_mismatch_threshold_lengths() public {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::290 => for (uint i = 0; i < prices_august.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::301 => for (uint i = 0; i < prices_september.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::312 => for (uint i = 0; i < prices_october.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::350 => for (uint i = 0; i < simulatedResults.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::375 => for (uint i = 0; i < simulatedResults.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::429 => for (uint i = 0; i < simulatedResults.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::438 => for (uint i = 0; i < prices_august.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::444 => for (uint i = 0; i < prices_september.length; ++i) {
2023-09-ondo/forge-tests/rwaOracles/RWADynamicOracle.t.sol::450 => for (uint i = 0; i < prices_october.length; ++i) {
```
## [G-03] Use not equal to 0 instead of Greater than 0 for unsigned integer comparison
Description
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.
```txt
023-09-ondo/contracts/bridge/DestinationBridge.sol::159 => if (t.approvers.length > 0) {
2023-09-ondo/contracts/usdy/rUSDY.sol::435 => require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
2023-09-ondo/contracts/usdy/rUSDY.sol::450 => require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
```
## [G-04] Use immutable for openzeppelin access controls roles declarations
Description
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::99 => if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {
2023-09-ondo/contracts/bridge/DestinationBridge.sol::108 => bytes32 txnHash = keccak256(payload);
2023-09-ondo/contracts/bridge/DestinationBridge.sol::195 => * @param txnHash The keccak256 hash of the payload
2023-09-ondo/contracts/bridge/DestinationBridge.sol::238 => chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));
2023-09-ondo/contracts/usdy/rUSDY.sol::97 => bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
2023-09-ondo/contracts/usdy/rUSDY.sol::98 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
2023-09-ondo/contracts/usdy/rUSDY.sol::99 => bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
2023-09-ondo/contracts/usdy/rUSDY.sol::100 => bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
2023-09-ondo/contracts/usdy/rUSDY.sol::102 => keccak256("LIST_CONFIGURER_ROLE");
```
## [G-05] Long revert strings
Description
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::18 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IAxelarGateway.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::19 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IAxelarGasService.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::20 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/axelar/AxelarExecutable.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::21 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IRWALike.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::22 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IAllowlist.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::23 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts/access/Ownable.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::24 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts/security/Pausable.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::25 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/bridge/MintRateLimiter.sol";
2023-09-ondo/contracts/bridge/SourceBridge.sol::6 => import "../interfaces/IAxelarGasService.sol";
2023-09-ondo/contracts/bridge/SourceBridge.sol::9 => import {AddressToString} from "../external/axelar/StringAddressUtils.sol";
2023-09-ondo/contracts/bridge/SourceBridge.sol::10 => import "../external/openzeppelin/contracts/access/Ownable.sol";
2023-09-ondo/contracts/bridge/SourceBridge.sol::11 => import "../external/openzeppelin/contracts/security/Pausable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::18 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::19 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20MetadataUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::20 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/proxy/Initializable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::21 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::22 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::23 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::24 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/usdy/blocklist/BlocklistClientUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::25 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/usdy/allowlist/AllowlistClientUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::26 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/sanctions/SanctionsListClientUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::27 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IUSDY.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::28 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/rwaOracles/IRWADynamicOracle.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::307 => require(currentAllowance >= _amount, "TRANSFER_AMOUNT_EXCEEDS_ALLOWANCE");
2023-09-ondo/contracts/usdy/rUSDY.sol::435 => require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
2023-09-ondo/contracts/usdy/rUSDY.sol::450 => require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
2023-09-ondo/contracts/usdy/rUSDY.sol::635 => require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");
2023-09-ondo/contracts/usdy/rUSDY.sol::638 => "rUSDY: 'sender' address not on allowlist"
2023-09-ondo/contracts/usdy/rUSDY.sol::646 => require(_isAllowed(from), "rUSDY: 'from' address not on allowlist");
2023-09-ondo/contracts/usdy/rUSDY.sol::653 => require(_isAllowed(to), "rUSDY: 'to' address not on allowlist");
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::20 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/Proxy.sol";
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::21 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/usdy/rUSDY.sol";
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::22 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IMulticall.sol";
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::155 => require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");
```
## [G-06] Use shift right or left instead of division or multiplication if possible
Description
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::18 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IAxelarGateway.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::19 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IAxelarGasService.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::20 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/axelar/AxelarExecutable.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::21 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IRWALike.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::22 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IAllowlist.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::23 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts/access/Ownable.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::24 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts/security/Pausable.sol";
2023-09-ondo/contracts/bridge/DestinationBridge.sol::25 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/bridge/MintRateLimiter.sol";
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::18 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/rwaOracles/IRWAOracle.sol";
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::19 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts/access/AccessControlEnumerable.sol";
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::20 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts/security/Pausable.sol";
2023-09-ondo/contracts/rwaOracles/RWADynamicOracle.sol::343 => uint256 private constant ONE = 10 ** 27;
023-09-ondo/contracts/usdy/rUSDY.sol::18 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::19 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20MetadataUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::20 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/proxy/Initializable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::21 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::22 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::23 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::24 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/usdy/blocklist/BlocklistClientUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::25 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/usdy/allowlist/AllowlistClientUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::26 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/sanctions/SanctionsListClientUpgradeable.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::27 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IUSDY.sol";
2023-09-ondo/contracts/usdy/rUSDY.sol::28 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/rwaOracles/IRWADynamicOracle.sol";
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::19 => import {ProxyAdmin} from '/Users/williamsmith/Downloads/2023-09-ondo/contracts/external/openzeppelin/contracts/proxy/ProxyAdmin.sol';
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::20 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/Proxy.sol";
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::21 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/usdy/rUSDY.sol";
2023-09-ondo/contracts/usdy/rUSDYFactory.sol::22 => import "/Users/williamsmith/Downloads/2023-09-ondo/contracts/interfaces/IMulticall.sol";
```