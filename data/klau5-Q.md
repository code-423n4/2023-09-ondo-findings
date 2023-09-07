# `TransferShares` event parameter is not correct

## Links to affected code

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L439](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L439)

## Impact

Emit `TransferShares`  with wrong value at `wrap` function.

## Proof of Concept

`TransferShares` should emit `sharesValue` . But at `wrap` function, `TransferShares` emits `_USDYAmount` .

```solidity
event TransferShares(
  address indexed from,
  address indexed to,
  uint256 sharesValue
);

function wrap(uint256 _USDYAmount) external whenNotPaused {
  require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
  _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
  usdy.transferFrom(msg.sender, address(this), _USDYAmount);
  emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
  emit TransferShares(address(0), msg.sender, _USDYAmount);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function wrap(uint256 _USDYAmount) external whenNotPaused {
  require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
  _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
  usdy.transferFrom(msg.sender, address(this), _USDYAmount);
  emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
- emit TransferShares(address(0), msg.sender, _USDYAmount);
+ emit TransferShares(address(0), msg.sender, _USDYAmount * BPS_DENOMINATOR);
}
```

# Remove useless inheritance

## Links to affected code

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L21](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L21)

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L59](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L59)

## Impact

Imports and inherits useless contract

## Proof of Concept

rUSDY contract imports and inherits ContextUpgradeable contract, but there is no usage of this. There is no code using `_msgData()` or `_msgSender()` . This only increase deploy gas, so it would be better to remove unused import/inheritance.

```solidity
import "contracts/external/openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol";

contract rUSDY is
  Initializable,
  ContextUpgradeable,
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Remove import and inheritance of ContextUpgradeable.

# Remove unused role

## Links to affected code

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L98](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L98)

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L44](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L44)

## Impact

Unused roles are defined and granted. 

## Proof of Concept

There are some roles which is defined/granted but not used. 

### rUSDY

- MINTER_ROLE

### rUSDYFactory

This contract does not use role based access control.

- DEFAULT_ADMIN_ROLE

## Tools Used

Manual Review

## Recommended Mitigation Steps

Remove useless role and grant codes.

# `rUSDYDeployed` event parameter is not correct

## Links to affected code

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L105](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L105)

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L195](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L195)

## Impact

At `rUSDYDeployed` event emitting, name of rUSDY token is not same with real rUSDY name.

## Proof of Concept

`rUSDYDeployed` emits `Ondo Rebasing U.S. Dollar Yield` as rUSDY name.

```solidity
emit rUSDYDeployed(
  address(rUSDYProxy),
  address(rUSDYProxyAdmin),
  address(rUSDYImplementation),
  "Ondo Rebasing U.S. Dollar Yield",
  "rUSDY"
);
```

But the rUSDY token name is `Rebasing Ondo U.S. Dollar Yield`

```solidity
function name() public pure returns (string memory) {
  return "Rebasing Ondo U.S. Dollar Yield";
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
emit rUSDYDeployed(
  address(rUSDYProxy),
  address(rUSDYProxyAdmin),
  address(rUSDYImplementation),
- "Ondo Rebasing U.S. Dollar Yield",
+ "Rebasing Ondo U.S. Dollar Yield",
  "rUSDY"
);
```

# Remove useless error

## Links to affected code

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L334](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L334)

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L336](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L336)

## Impact

Useless error defined

## Proof of Concept

The `InvalidPrice` error and `PriceNotSet` error is defined but never used at RWADynamicOracle contract.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Remove `InvalidPrice` and `PriceNotSet` error at RWADynamicOracle contract.

# Role name and hash string do not match

## Links to affected code

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L97](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L97)

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L100](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L100)

## Impact

The role and hashed string is not same at rUSDY contract.

## Proof of Concept

```solidity
bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
```

The role and hashed string is not same at rUSDY contract. This could make confusing.

And `USDY_MANAGER_ROLE` should be changed to `RUSDY_MANAGER_ROLE`

## Tools Used

Manual Review

## Recommended Mitigation Steps

Modify role name or hashed string.

# preRebaseTokenAmount and postRebaseTokenAmount will be same value at SharesBurnt event

## Links to affected code

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L586-L599](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L586-L599)

[https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L397-L399](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L397-L399)

## Impact

`preRebaseTokenAmount` and `postRebaseTokenAmount` will be same value at SharesBurnt event

## Proof of Concept

`SharesBurnt` event emits `preRebaseTokenAmount` and `postRebaseTokenAmount` . But this two values are always same because `oracle.getPrice()` is not changed.

```solidity
/**
 * @notice An executed `burnShares` request
 *
 * @dev Reports simultaneously burnt shares amount
 * and corresponding rUSDY amount.
 * The shares amount is calculated twice: before and after the burning incurred rebase.
 *
 * @param account holder of the burnt shares
 * @param preRebaseTokenAmount amount of rUSDY the burnt shares (USDY) corresponded to before the burn
 * @param postRebaseTokenAmount amount of rUSDY the burnt shares (USDY) corresponded to after the burn
 * @param sharesAmount amount of burnt shares
 */
event SharesBurnt(
  address indexed account,
  uint256 preRebaseTokenAmount,
  uint256 postRebaseTokenAmount,
  uint256 sharesAmount
);

function _burnShares(
  address _account,
  uint256 _sharesAmount
) internal whenNotPaused returns (uint256) {
  ...
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
  ...
}

function getRUSDYByShares(uint256 _shares) public view returns (uint256) {
  return (_shares * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Emit just `tokenAmount` instead of `preRebaseTokenAmount` and `postRebaseTokenAmount` .