### 1. Any comments for the judge to contextualize your findings
- Focused on the rUSDY and Destination Bridge contract, and followed through with all the test files for those contracts

### 2. Approach taken in evaluating the codebase
- Understood the relation between USDY,rUSDY and shares first, then check all the functions in rUSDY to make sure that they are working as intended and cannot be exploited easily
- Run the test files of rUSDY and run some fuzz testing to make sure that wrap() and unwrap() works as intended with different time stamp and oracle price
- Moved on to the source and destination bridge and ran the tests 
- Made sure that the test are properly written and focused on the setThresholds() function in Destination contract
- Zoomed out and looked at all the contracts as a whole to check whether all contracts and imports are written correctly, and all contracts have the pause function as well as being protected by the modifier.

### 3. Mechanism review
- I don't think that rUSDY can be called a rebasing token because it is not pegged to USDY but rather to the dollar. Also, the price of rUSDY is taken from the oracle of USDY, so it's more like a vault token than a rebasing token
- There is a part in the rUSDY contract that has a truncation to zero issue. If the shares passed into getRUSDYByShares() is less than 10,000 and the oracle returns the price of 1e18, then the price of RUSDYByShares will be zero. Minimal loss as only dust amounts will be lost 

```
  function getRUSDYByShares(uint256 _shares) public view returns (uint256) {
    return (_shares * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);
  }
```
- In the Destination contract, there is a heavy emphasis on restricting users from minting the token on the destination chain. The restriction includes having the need to have approvals and having a rate limiter with a maxTokenlimit 
```
      _checkAndUpdateInstantMintLimit(txn.amount);
```

### 4. Centralization Risk

- The owner can burn the rUSDY of any account and pocket the USDY
```
  function burn(
    address _account,
    uint256 _amount
  ) external onlyRole(BURNER_ROLE) {
    uint256 sharesAmount = getSharesByRUSDY(_amount);
    _burnShares(_account, sharesAmount);
    usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);
```
- Owner controls the approval and mint limit of the Destination contract as well
```
  function addApprover(address approver) external onlyOwner {
    approvers[approver] = true;
    emit ApproverAdded(approver);
  }
```
- Owner is able to pause and unpause all parts of the protocol
```
  function pause() external onlyOwner {
    _pause();
  }
  /**
   * @notice Admin function to unpause the contract
   *
   * @dev Only used for bridge functions
   */
  function unpause() external onlyOwner {
    _unpause();
```
- In summary, if owner turns malicious, the whole contract is at stake

### Time spent:
14 hours