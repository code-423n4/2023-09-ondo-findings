## Non Critical Issues

|       | Issue                                                                                    |
| ----- | :--------------------------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                     |
| NC-2  | ADD TO BLACKLIST FUNCTION                                                                |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                                 |
| NC-4  | SAME CONSTANT REDEFINED ELSEWHERE                                                        |
| NC-5  | IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED                                           |
| NC-6  | MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL                                   |
| NC-7  | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT |
| NC-8  | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS                             |
| NC-9  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                            |
| NC-10 | NO SAME VALUE INPUT CONTROL                                                              |
| NC-11 | Add parameter to event-emit                                                              |
| NC-12 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                                       |
| NC-13 | LINES ARE TOO LONG                                                                       |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

255:   function setThresholds(

286:   function setMintLimit(uint256 mintLimit) external onlyOwner {

287:     _setMintLimit(mintLimit);

295:   function setMintLimitDuration(uint256 mintDuration) external onlyOwner {

296:     _setMintLimitDuration(mintDuration);

343:         ALLOWLIST.setAccountStatus(

```

```solidity
File: contracts/bridge/SourceBridge.sol

121:   function setDestinationChainContractAddress(

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

32:     address setter,

41:     _grantRole(SETTER_ROLE, setter);

151:   function setRange(

```

```solidity
File: contracts/usdy/rUSDY.sol

662:   function setOracle(address _oracle) external onlyRole(USDY_MANAGER_ROLE) {

698:   function setBlocklist(

701:     _setBlocklist(blocklist);

709:   function setAllowlist(

712:     _setAllowlist(allowlist);

720:   function setSanctionsList(

723:     _setSanctionsList(sanctionsList);

```

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] SAME CONSTANT REDEFINED ELSEWHERE

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

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

28:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

```solidity
File: contracts/usdy/rUSDY.sol

99:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

### [NC-5] IMPLEMENTATION CONTRACT MAY NOT BE INITIALIZED

#### Description:

OpenZeppelin recommends that the initializer modifier be applied to constructors.

Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDY.sol

20: import "contracts/external/openzeppelin/contracts-upgradeable/proxy/Initializable.sol";

58:   Initializable,

```

### [NC-6] MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

#### Description:

If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}\_init function, not the parent public initialize(...) function.

External instead of public would give more sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDY.sol

109:   function initialize(

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

90:     rUSDYProxied.initialize(

```

### [NC-7] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### Description:

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

WConstants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

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

### [NC-8] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions.

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDY.sol

109:   function initialize(

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

90:     rUSDYProxied.initialize(

```

### [NC-9] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

16: pragma solidity 0.8.16;

```

```solidity
File: contracts/bridge/SourceBridge.sol

1: pragma solidity 0.8.16;

```

```solidity
File: contracts/rwaOracles/IRWADynamicOracle.sol

16: pragma solidity 0.8.16;

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

16: pragma solidity 0.8.16;

```

```solidity
File: contracts/usdy/rUSDY.sol

16: pragma solidity 0.8.16;

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

16: pragma solidity 0.8.16;

```

### [NC-10] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDYFactory.sol

52:     guardian = _guardian;

```

### [NC-11] Add parameter to event-emit

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

The events should include the new value and old value where possible.

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

212:     emit ApproverAdded(approver);

222:     emit ApproverRemoved(approver);

```

### [NC-12] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts

 solidity: {
    compilers: [
      {
        version: "0.8.16",
        settings: {
          optimizer: {
            enabled: true,
            runs: 100,
          },
        },
      },
      {
        version: "0.7.0",
        settings: {
          optimizer: {
            enabled: true,
            runs: 100,
          },
        },
      },
      {
        version: "0.6.12",
        settings: {
          optimizer: {
            enabled: true,
            runs: 100,
          },
        },
      },
      {
        version: "0.4.24",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    ],
  },
```

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

### [NC-13] LINES ARE TOO LONG

#### Description:

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

7: ██  ██ ╒█▀'   ╙█▌ ╙█▌ ██     ▐██      ███  █████,  ██  ██▌    └██▌  ██▌     └██▌

8: ██ ▐█▌ ██      ╟█  █▌ ╟█     ██▌      ▐██  ██ └███ ██  ██▌     ╟██ j██       ╟██

9: ╟█  ██ ╙██    ▄█▀ ▐█▌ ██     ╙██      ██▌  ██   ╙████  ██▌    ▄██▀  ██▌     ,██▀

238:     chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));

414:   event ThresholdSet(string chain, uint256[] amounts, uint256[] numOfApprovers);

```

```solidity
File: contracts/bridge/SourceBridge.sol

7: import {AddressToString} from "contracts/external/axelar/StringAddressUtils.sol";

```

```solidity
File: contracts/rwaOracles/IRWADynamicOracle.sol

7: ██  ██ ╒█▀'   ╙█▌ ╙█▌ ██     ▐██      ███  █████,  ██  ██▌    └██▌  ██▌     └██▌

8: ██ ▐█▌ ██      ╟█  █▌ ╟█     ██▌      ▐██  ██ └███ ██  ██▌     ╟██ j██       ╟██

9: ╟█  ██ ╙██    ▄█▀ ▐█▌ ██     ╙██      ██▌  ██   ╙████  ██▌    ▄██▀  ██▌     ,██▀

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

7: ██  ██ ╒█▀'   ╙█▌ ╙█▌ ██     ▐██      ███  █████,  ██  ██▌    └██▌  ██▌     └██▌

8: ██ ▐█▌ ██      ╟█  █▌ ╟█     ██▌      ▐██  ██ └███ ██  ██▌     ╟██ j██       ╟██

9: ╟█  ██ ╙██    ▄█▀ ▐█▌ ██     ╙██      ██▌  ██   ╙████  ██▌    ▄██▀  ██▌     ,██▀

19: import "contracts/external/openzeppelin/contracts/access/AccessControlEnumerable.sol";

```

```solidity
File: contracts/usdy/rUSDY.sol

7: ██  ██ ╒█▀'   ╙█▌ ╙█▌ ██     ▐██      ███  █████,  ██  ██▌    └██▌  ██▌     └██▌

8: ██ ▐█▌ ██      ╟█  █▌ ╟█     ██▌      ▐██  ██ └███ ██  ██▌     ╟██ j██       ╟██

9: ╟█  ██ ╙██    ▄█▀ ▐█▌ ██     ╙██      ██▌  ██   ╙████  ██▌    ▄██▀  ██▌     ,██▀

18: import "contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

19: import "contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20MetadataUpgradeable.sol";

20: import "contracts/external/openzeppelin/contracts-upgradeable/proxy/Initializable.sol";

21: import "contracts/external/openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol";

22: import "contracts/external/openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";

23: import "contracts/external/openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";

117:     __rUSDY_init(blocklist, allowlist, sanctionsList, _usdy, guardian, _oracle);

227:     return (_sharesOf(_account) * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);

245:   function transfer(address _recipient, uint256 _amount) public returns (bool) {

635:       require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

7: ██  ██ ╒█▀'   ╙█▌ ╙█▌ ██     ▐██      ███  █████,  ██  ██▌    └██▌  ██▌     └██▌

8: ██ ▐█▌ ██      ╟█  █▌ ╟█     ██▌      ▐██  ██ └███ ██  ██▌     ╟██ j██       ╟██

9: ╟█  ██ ╙██    ▄█▀ ▐█▌ ██     ╙██      ██▌  ██   ╙████  ██▌    ▄██▀  ██▌     ,██▀

```

## Low Issues

|     | Issue                                                             |
| --- | :---------------------------------------------------------------- |
| L-1 | Initializing state-variables in proxy-based upgradeable contracts |
| L-2 | Avoid `transfer()`/`send()` as reentrancy mitigations             |
| L-3 | The critical parameters in initialize(...) are not set safely     |
| L-4 | UNIFY RETURN CRITERIA                                             |
| L-5 | REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR                        |
| L-6 | Timestamp Dependence                                              |
| L-7 | Account existence check for low-level calls                       |
| L-8 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`                          |

### [L-1] Initializing state-variables in proxy-based upgradeable contracts

#### Description:

This should be done in initializer functions and not as part of the state variable declarations in which case they won’t be set.

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDY.sol

109:   function initialize(

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

90:     rUSDYProxied.initialize(

```

### [L-2] Avoid `transfer()`/`send()` as reentrancy mitigations

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

324:     IRWALike(_token).transfer(owner(), balance);

```

```solidity
File: contracts/usdy/rUSDY.sol

454:     usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);

680:     usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-3] The critical parameters in initialize(...) are not set safely

#### **Proof Of Concept**

```solidity
File: contracts/usdy/rUSDY.sol

109:   function initialize(

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

90:     rUSDYProxied.initialize(

```

### [L-4] UNIFY RETURN CRITERIA

#### Description:

In files sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

177:   function _checkThresholdMet(bytes32 txnHash) internal view returns (bool) {

361:   function getNumApproved(bytes32 txnHash) external view returns (uint256) {

```

```solidity
File: contracts/bridge/SourceBridge.sol

162:   ) external payable override onlyOwner returns (bytes[] memory results) {

```

```solidity
File: contracts/rwaOracles/IRWADynamicOracle.sol

20:   function getPrice() external view returns (uint256);

```

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

61:     returns (uint256 price, uint256 timestamp)

75:   function getPrice() public view whenNotPaused returns (uint256 price) {

110:   ) external view returns (uint256 price) {

265:   ) internal pure returns (uint256 price) {

282:   function roundUpTo8(uint256 value) internal pure returns (uint256) {

349:   ) internal pure returns (uint256 z) {

400:   function _rmul(uint256 x, uint256 y) internal pure returns (uint256 z) {

404:   function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {

```

```solidity
File: contracts/usdy/rUSDY.sol

194:   function name() public pure returns (string memory) {

202:   function symbol() public pure returns (string memory) {

209:   function decimals() public pure returns (uint8) {

216:   function totalSupply() public view returns (uint256) {

226:   function balanceOf(address _account) public view returns (uint256) {

245:   function transfer(address _recipient, uint256 _amount) public returns (bool) {

259:   ) public view returns (uint256) {

276:   function approve(address _spender, uint256 _amount) public returns (bool) {

305:   ) public returns (bool) {

330:   ) public returns (bool) {

356:   ) public returns (bool) {

372:   function getTotalShares() public view returns (uint256) {

381:   function sharesOf(address _account) public view returns (uint256) {

390:   ) public view returns (uint256) {

397:   function getRUSDYByShares(uint256 _shares) public view returns (uint256) {

419:   ) public returns (uint256) {

500:   function _sharesOf(address _account) internal view returns (uint256) {

546:   ) internal whenNotPaused returns (uint256) {

578:   ) internal whenNotPaused returns (uint256) {

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

81:   ) external onlyGuardian returns (address, address, address) {

128:   ) external payable override onlyGuardian returns (bytes[] memory results) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-5] REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR

#### Description:

The correct and clear error description explains to the user why the function reverts, but the error descriptions below in the project are not self-explanatory.

#### **Proof Of Concept**

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

377:             revert(0, 0)

381:             revert(0, 0)

387:               revert(0, 0)

391:               revert(0, 0)

405:     require(y == 0 || (z = x * y) / y == x);

```

#### Recommended Mitigation Steps:

Error definitions should be added to the require block, not exceeding 32 bytes or we should use custom errors

### [L-6] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

#### **Proof Of Concept**

```solidity
File: contracts/rwaOracles/RWADynamicOracle.sol

64:     timestamp = block.timestamp;

79:       if (range.start <= block.timestamp) {

80:         if (range.end <= block.timestamp) {

83:           return derivePrice(range, block.timestamp);

```

### [L-7] Account existence check for low-level calls

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

```solidity
File: contracts/bridge/SourceBridge.sol

165:       (bool success, bytes memory ret) = address(exCallData[i].target).call{

```

```solidity
File: contracts/usdy/rUSDYFactory.sol

131:       (bool success, bytes memory ret) = address(exCallData[i].target).call{

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-8] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

```solidity
File: contracts/bridge/DestinationBridge.sol

324:     IRWALike(_token).transfer(owner(), balance);

```

```solidity
File: contracts/usdy/rUSDY.sol

454:     usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);

680:     usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);

```

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.
