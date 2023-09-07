# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | `preRebaseTokenAmount` and `postRebaseTokenAmount` will both have the same value which is misleading | Low | 1 |
| 2 | Wrong fields emitted in `wrap` function events | Low | 2 |
| 3 | `MINTER_ROLE` not usable as there is no external/public mint function | NC | 1 |
| 4 | Comments at the top of `USDYFactory` are wrong | NC | 1 |


## Findings

### 1- `preRebaseTokenAmount` and `postRebaseTokenAmount` will both have the same value which is misleading :

#### Risk : Low

In the `_burnShares` function, there two values that are computed `preRebaseTokenAmount` and `postRebaseTokenAmount` which represent the RUSDY amount before and after the change in the value of `totalShares` : 

```
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

    ...
}
```

The issue is that those two values will have exactly the same value as they are both obtained from the `getRUSDYByShares` function at the same block timestamp :

```
function getRUSDYByShares(uint256 _shares) public view returns (uint256) {
    return (_shares * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
}
```

As you can see the value returned from `getRUSDYByShares` doesn't depend on the `totalShares` change and the only thing that can change between the two call to the `getRUSDYByShares` function is `oracle.getPrice()`, but by taking a look at the `getPrice()` function in the `RWADynamicOracle` contract we can notice that the price will remain the same if fetched at the same block timestamp, and thus the values `preRebaseTokenAmount` and `postRebaseTokenAmount` will be equal.

As both those values are emitted in the `SharesBurnt` event, this issue will be misleading for all the outside parties as it seems to indicate that the RUSDY token does not rebase (remain the same when shares are burned) which is wrong and it could lead to bad accounting in their apps, UI or protocols.

#### Mitigation

I recommend to review the logic of the `_burnShares` function and check if the fact that `preRebaseTokenAmount` and `postRebaseTokenAmount` have the same value is correct or not.


### 2- Wrong fields emitted in `wrap` function events :

#### Risk : Low

In the `wrap` function the wrong amounts are emitted in both `Transfer` and `TransferShares` events, the issue is highlighted in the code below : 

```
function wrap(uint256 _USDYAmount) external whenNotPaused {
    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
    _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
    // @audit should be Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR));
    emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
    // @audit should be TransferShares(address(0), msg.sender, _USDYAmount * BPS_DENOMINATOR)
    emit TransferShares(address(0), msg.sender, _USDYAmount);
}
```

In the `Transfer` event the amount emitted is `getRUSDYByShares(_USDYAmount)` but this is incorrect because `_USDYAmount` does not represent the true shares amount and thus the `getRUSDYByShares` function will return the wrong rUSDY amount, the true shares amount should be `_USDYAmount * BPS_DENOMINATOR`.

The same issue occurs in the `TransferShares` event where `_USDYAmount` is emitted as the number of shares when it should be `_USDYAmount * BPS_DENOMINATOR`.

I submit this issue as Low risk because it will not affect the code logic directly or lead to a loss of funds but it could be considered as a valid medium (with lower impact) as it will still cause problems for all the parties that rely on those emitted events in their apps or protocols.

#### Mitigation
Emit the correct amount values in both `Transfer` and `TransferShares` events in the `wrap` function : 

```
function wrap(uint256 _USDYAmount) external whenNotPaused {
    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
    _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
    // @audit Emit correct amounts
    emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR));
    emit TransferShares(address(0), msg.sender, _USDYAmount * BPS_DENOMINATOR);
}
```


### 3- `MINTER_ROLE` not usable as there is not external mint function :

#### Risk : Non critical

The `MINTER_ROLE` is setup in the `__rUSDY_init_unchained` function but this role is not used inside the contract for any access control task as there is not external/public function that allow to mint new shares to other users and the only way to do so is by calling the `wrap` function. 

Either consider adding a mint function that allow the `MINTER_ROLE` to mint new shares or remove these role from the contract.


### 4- Comments at the top of `USDYFactory` are wrong :

#### Risk : Non critical

The comments at the top of the `USDYFactory` contract contains incorrect information especially the following statement : 

```
ii) Revoke the `MINTER_ROLE`, `PAUSER_ROLE` & `DEFAULT_ADMIN_ROLE` from address(this).
```

As we can see in the `USDYFactory` contract there are no such actions performed in any function, so either this logic was forgotten in which case it must be added or if the comments are just wrong/outdated then they should be updated to avoid misleading readers.
