# Low Risk Issues

## 1. ShareBurnt event is emitted with two identical inputs.

```preRebaseTokenAmount``` and ```postRebaseTokenAmount``` calculations will always return the same value in the ```_burnShares``` function as they take the same input parameter ```_sharesAmount``` and they share an identical timestamp (used in ```oracle.getPrice()```), while the range was not changed.

```solidity
function _burnShares(
address _account,
uint256 _sharesAmount
) internal whenNotPaused returns (uint256) {
require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

_beforeTokenTransfer(_account, address(0), _sharesAmount);

uint256 accountShares = shares[_account];
require(_sharesAmount <= accountShares, "BURN_AMOUNT_EXCEEDS_BALANCE");

uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

totalShares -= _sharesAmount;

shares[_account] = accountShares - _sharesAmount;

uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

emit SharesBurnt(
_account,
preRebaseTokenAmount,
postRebaseTokenAmount,
_sharesAmount
);

return totalShares;

// Notice: we're not emitting a Transfer event to the zero address here since shares burn
// works by redistributing the amount of tokens corresponding to the burned shares between
// all other token holders. The total supply of the token doesn't change as the result.
// This is equivalent to performing a send from `address` to each other token holder address,
// but we cannot reflect this as it would require sending an unbounded number of events.

// We're emitting `SharesBurnt` event to provide an explicit rebase log record nonetheless.
}
```

Given [getRUSDYByShares](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L397) definition below:

```solidity
function getRUSDYByShares(uint256 _shares) public view returns (uint256) {
return (_shares * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
}
```

And the getPrice() function in [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L75)

```solidity
function getPrice() public view whenNotPaused returns (uint256 price) {
uint256 length = ranges.length;
for (uint256 i = 0; i < length; ++i) {
Range storage range = ranges[(length - 1) - i];
if (range.start <= block.timestamp) {
if (range.end <= block.timestamp) {
return derivePrice(range, range.end - 1);
} else {
return derivePrice(range, block.timestamp);
}
}
}
}
```


## 2. Imports should be named explicitly.

This is good practice for several reasons:

- this gives more control about what is imported into each file
- this is less prone to naming conflicts as it is clear what objects are imported as opposed to entire files.
- this is easier for external user to read and to find where each contract has been imported

There is 32 occurrences of unnamed imports in this scope:

[DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol)
```solidity
import "contracts/interfaces/IAxelarGateway.sol";
import "contracts/interfaces/IAxelarGasService.sol";
import "contracts/external/axelar/AxelarExecutable.sol";
import "contracts/interfaces/IRWALike.sol";
import "contracts/interfaces/IAllowlist.sol";
import "contracts/external/openzeppelin/contracts/access/Ownable.sol";
import "contracts/external/openzeppelin/contracts/security/Pausable.sol";
import "contracts/bridge/MintRateLimiter.sol";
```

[SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol)
```solidity
import "contracts/interfaces/IAxelarGateway.sol";
import "contracts/interfaces/IAxelarGasService.sol";
import "contracts/interfaces/IMulticall.sol";
import "contracts/interfaces/IRWALike.sol";
```
and
```solidity
import "contracts/external/openzeppelin/contracts/access/Ownable.sol";
import "contracts/external/openzeppelin/contracts/security/Pausable.sol";
```

[RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol)
```solidity
import "contracts/rwaOracles/IRWAOracle.sol";
import "contracts/external/openzeppelin/contracts/access/AccessControlEnumerable.sol";
import "contracts/external/openzeppelin/contracts/security/Pausable.sol";
```

[rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol)
```solidity
import "contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
import "contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20MetadataUpgradeable.sol";
import "contracts/external/openzeppelin/contracts-upgradeable/proxy/Initializable.sol";
import "contracts/external/openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol";
import "contracts/external/openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
import "contracts/external/openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
import "contracts/usdy/blocklist/BlocklistClientUpgradeable.sol";
import "contracts/usdy/allowlist/AllowlistClientUpgradeable.sol";
import "contracts/sanctions/SanctionsListClientUpgradeable.sol";
import "contracts/interfaces/IUSDY.sol";
import "contracts/rwaOracles/IRWADynamicOracle.sol";
```
[rUSDYFactory.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol)
```solidity
import "contracts/external/openzeppelin/contracts/proxy/ProxyAdmin.sol";
import "contracts/Proxy.sol";
import "contracts/usdy/rUSDY.sol";
import "contracts/interfaces/IMulticall.sol";
```

### Mitigation steps:
Explicitly name all imports actually used in the contract.


## 3. All contract variables should be declared as private with associated getter functions.

This enables for more control on which aspect of the data developers want to expose.

There are 31 occurrences of this in scope.

### Mitigation steps:

Switch public variables to private visibility and define getter functions for the ones that are meant to be exposed.
e.g. in https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97

```solidity
bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
```

Should become

```solidity
bytes32 private constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");

function getManagerRole() external view returns(bytes32){
    return USDY_MANAGER_ROLE;
}
```


## 4. Variable declarations should be grouped by visibility and separated from error declarations

In [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol), the ```UnwrapTooSmall()``` error is defined in between two public variable declarations.

```solidity
// Used to scale up usdy amount -> shares
uint256 public constant BPS_DENOMINATOR = 10_000;

// Error when redeeming shares < `BPS_DENOMINATOR`
error UnwrapTooSmall();

/// @dev Role based access control roles
bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
```

### Mitigation steps:
Group all error declarations in the same part of the contract to increase readability.

## 5. Immutable variables should use the “i_” prefix to distinguish them from constant variables that use uppercase names.

This is good practice to avoid confusion between constant variables and immutable variables, and to thus improve readability of the contract.

There are 6 occurrences of this in scope.

[DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L34)

```solidity
/// @notice Token contract bridged by this contract
IRWALike public immutable TOKEN;

/// @notice Pointer to AxelarGateway contract
IAxelarGateway public immutable AXELAR_GATEWAY;

/// @notice Pointer to USDY allowlist
IAllowlist public immutable ALLOWLIST;
```

[SourceBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L18)

```solidity
/// @notice Token contract bridged by this contract
IRWALike public immutable TOKEN;

/// @notice Pointer to AxelarGateway contract
IAxelarGateway public immutable AXELAR_GATEWAY;

/// @notice Pointer to AxelarGasService contract
IAxelarGasService public immutable GAS_RECEIVER;
```

### Mitigation steps:

Prefix all immutable variables with “i_” 
E.g. in https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L40

```solidity
IAllowlist public immutable ALLOWLIST;

```

Should be 

```solidity
IAllowlist public immutable i_allowList;

```

Note that these immutable variables should also be defined as private with associated getter functions as mentioned in 3.

## 6. Inaccurate ordering of functions and misleading comments:

```function _mintIfThresholdMet(bytes32 txnHash) internal {```

([DestinationBridge.sol#L337](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L337)) is defined under a section labeled as 

```solidity
/*//////////////////////////////////////////////////////////////
Public Functions
//////////////////////////////////////////////////////////////*/
```

### Mitigation steps:

If categorization of function visibility is made explicit in comment, visibility of functions should match the section they are in (i.e. for example ```_mintIfThresholdMet``` should be declared under the “internal functions” section)

## 7. Errors defined but not used

There are 2 occurrences of unused errors in the scope.

```solidity
error InvalidPrice();
```
In [RWADynamicOracle.sol#L334](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L334)

```solidity
error PriceNotSet();
```
In [RWADynamicOracle.sol#L336](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L336)

Are defined but not used

### Mitigation steps: 
Use errors where relevant or delete unused error declarations.


## 8. Public functions that are not used within the contract itself should be marked as external instead.

There are 4 occurrences in [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol): ```name()```, ```symbol()```, ```decimals()```, ```totalSupply()``` are all unused within the contract.

### Mitigation steps: 
Mark all public functions not used in the contract as external to improve clarity of the codebase.

## 9. Ordering of events, errors and functions by visibility is not consistent across files.

This makes it less straightforward for external reviewers to eyeball where these objects are defined.

E.g. errors and events are defined at the end of the contract in [DestinationBridge.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol) and [RWADynamicOracle.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol) (before helper functions for the latter) but at the start / in the middle for [rUSDY.sol](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol)

### Mitigation steps:

Unless there is a clear readability improvement from not doing so, define objects in the same order across files.

## 10. Typos

We found a few typos in comments, namely 2 occurrences:
In comments: 2 occurrences: 
[rUSDY.sol#L631](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L631) “facliitate” 
[rUSDY.sol#L658-659](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L658)
```solidity 
@notice Sets the Oracle address
* @dev The new oracle must comply with the `IPricerReader` interface
```
“Oracle” or “oracle” inconsistency
