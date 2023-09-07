# 1
## Title
Unbounded loops used
## Impact
In the `setThresholds` function we see the function allows owner to input an unbounded length of arrays for amounts and numOfApprovers. This can lead to a long list of input which could cause the gasLimit to be hit and txn reverted with gas wasted.
## Proof of Concept
With amount array length of 15 and above, it is highly likely to hit the gasLimit
## Tools Used
Manual Review

## Recommended Mitigation Steps
Add a max length of array with mitigation steps to either revert or deal with the max limit of array length only

# 2
## Title
Use Maps rather than array
## Impact
We see the code `mapping(bytes32 => TxnThreshold) public txnToThresholdSet;` and 
```solidity
struct TxnThreshold {
    uint256 numberOfApprovalsNeeded;
    address[] approvers;
  } 
```
## Proof of Concept

## Tools Used
Manual Review

## Recommended Mitigation Steps
It could easily be update to `mapping(bytes32 => address => TxnThreshold)public txnToThresholdSet;` and 
```solidity
struct TxnThreshold {
    uint256 numberOfApprovalsNeeded;
    address approver;
  } 
```
and `mapping(bytes32 => number) TxnThreshold` for length of approvers, this is to avoid use of long arrays

# 3
## Title
Use an admin role instead of onlyOwner
## Impact
It is better to use admin or ownership roles to allow for inheritance and multiple access incase of wallet hacks and also to avoid a single point of failure.
## Proof of Concept
ownership can be stolen with wallet hacks, as seen by recent hacks
## Tools Used
Manual Review

## Recommended Mitigation Steps
It is better to use admin or ownership roles to allow for inheritance and multiple access incase of wallet hacks and also to avoid a single point of failure.

# 4
## Title
No check for number of approvers
## Impact
There is no check for the sorting of numOfApprovers in the `setThreshold` function, while the amount is checked to be sorted,there could be a scenario where `numOfApprovers` is not and this could lead to inconsistencies

## Proof of Concept
For example having [1000,2000,3000,400] as `amounts` and [2,4,3,5] as `numOfApprovers` would pass and cause inconsistencies where lower amounts would require more `numOfApprovers` than higher amounts.
## Tools Used
Manual Review

## Recommended Mitigation Steps
Add a check for `numOfApprovers` sorted as done for `amounts`