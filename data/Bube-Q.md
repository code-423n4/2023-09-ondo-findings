low[01]
The Daily Interest Rate is not validated

The contract `RWADynamicOracle.sol` does not validate the input parameter `dailyIR` (daily interest rate)
in constructor, which is used in `simulateRange` function. This could lead to incorrect price calculations if it set improperly.

There is 1 instance of this issue:

File: contracts/rwaOracles/RWADynamicOracle.sol:

```
constructor(
    address admin,
    address setter,
    address pauser,
    uint256 firstRangeStart,
    uint256 firstRangeEnd,
    uint256 dailyIR,
    uint256 startPrice
  ) {
    _grantRole(DEFAULT_ADMIN_ROLE, admin);
    _grantRole(PAUSER_ROLE, pauser);
    _grantRole(SETTER_ROLE, setter);

    if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
    uint256 trueStart = (startPrice * ONE) / dailyIR;
    ranges.push(Range(firstRangeStart, firstRangeEnd, dailyIR, trueStart));
  }
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L30-L46

Add checks to ensure that the dailyIR is within an expected range to prevent any potential issues.