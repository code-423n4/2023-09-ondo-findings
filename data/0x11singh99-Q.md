# Low-Level and Non-Critical issues

## Summary

### Low Level List

| Number | Issue Details                                                                                                      | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------- | :-------: |
| [L-01] | Address should be checked for 0 address before transferring to it.                                                                 |     1     |


### Non Critical List

| Number  | Issue Details                                                       | Instances |
| :-----: | :------------------------------------------------------------------ | :-------: |
| [NC-01] | Some Contracts are not following proper solidity style guide layout |     17     |



# LOW FINDINGS

## [L-01] Address should be checked for 0 address before transferring to it.

Check address is not zero before transferring to it. some tokens may not revert when tranferring to zero address.

**_1 Instance - 1 File_**

```solidity
File : contracts/usdy/rUSDY.sol

301:   function transferFrom(
    address _sender,
    address _recipient,
    uint256 _amount
  ) public returns (bool) {
    uint256 currentAllowance = allowances[_sender][msg.sender];
    require(currentAllowance >= _amount, "TRANSFER_AMOUNT_EXCEEDS_ALLOWANCE");

    _transfer(_sender, _recipient, _amount);///@audit check
    _approve(_sender, msg.sender, currentAllowance - _amount);
    return true;
312:  }

```
[301-312](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L301C1-L312C4)

# NON-CRITICAL FINDINGS

## [NC-01] Some Contracts are not following proper solidity style guide layout

/ Layout of Contract: According to solidity Docs
// version
// imports
// errors
// interfaces, libraries, contracts
// Type declarations
// State variables
// Events
// Modifiers
// Functions

// Layout of Functions:
// constructor
// receive function (if exists)
// fallback function (if exists)
// external
// public
// internal
// private
// internal & private view & pure functions
// external & public view & pure functions
https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout

**Note: These instances missed by bot report**

**_17 Instance - 3 Files_**

```solidity
File : contracts/bridge/DestinationBridge.sol

///@audit error should be declared in starting of contract
439: error NotApprover();
     error NoThresholdMatch();
     error ThresholdsNotInAscendingOrder();
     error ChainNotSupported();
     error SourceNotSupported();
     error NonceSpent();
     error AlreadyApproved();
     error InvalidVersion();
448: error ArrayLengthMismatch();
```
[439-448](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L439C3-L448C31)

```solidity
File : contracts/rwaOracles/RWADynamicOracle.sol

///@audit struct, events and errors should be declared in starting of contract

295:  struct Range {
    uint256 start;
    uint256 end;
    uint256 dailyInterestRate;
    uint256 prevRangeClosePrice;
  }

  event RangeSet(
    uint256 indexed index,
    uint256 start,
    uint256 end,
    uint256 dailyInterestRate,
    uint256 prevRangeClosePrice
  );

  event RangeOverriden(
    uint256 indexed index,
    uint256 newStart,
    uint256 newEnd,
    uint256 newDailyInterestRate,
    uint256 newPrevRangeClosePrice
  );

      error InvalidPrice();
      error InvalidRange();
336:  error PriceNotSet();

```
[295-336](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L295C2-L336C23)

```solidity
File : contracts/usdy/rUSDYFactory.sol

///@audit event and modifier should be declared above

146: event rUSDYDeployed(
    address proxy,
    address proxyAdmin,
    address implementation,
    string name,
    string ticker
  );

   modifier onlyGuardian() {
     require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");
     _;
157:  }

```
[146-157](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L146C3-L157C4)