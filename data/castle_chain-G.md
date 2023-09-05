# Gas optimization report 
# DestinationBridge 

## 1) unnecessary check inside the for loop that waste gas can be optimized . 
the function `setThresholds()` uses for loop to assign the thresholds to the amounts , the for loop always check 
```
  for (uint256 i = 0; i < amounts.length; ++i) {
      if (i == 0) {
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      }
```
, and this condition will be true only once in the loop , so it can be executed outside of the loop to avoid unnecessary check and save much gas . 
**the optimized code should be**
```diff
  function setThresholds(
    string calldata srcChain,
    uint256[] calldata amounts,
    uint256[] calldata numOfApprovers
  ) external onlyOwner {
    if (amounts.length != numOfApprovers.length) {
      revert ArrayLengthMismatch();
    }
    delete chainToThresholds[srcChain];

-  for (uint256 i = 0; i < amounts.length; ++i) {
-    if (i == 0) {
-      chainToThresholds[srcChain].push(
-        Threshold(amounts[i], numOfApprovers[i])
-     );
-    } else {
+      chainToThresholds[srcChain].push(
+       Threshold(amounts[0], numOfApprovers[0])
+     );
+    for (uint256 i = 1; i < amounts.length; ++i) {
        if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
          revert ThresholdsNotInAscendingOrder();
        }
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      }
    }
    emit ThresholdSet(srcChain, amounts, numOfApprovers);
  }

```
**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L255-L279


## 2) read from the storage and make a copy of the struct to the memory before the check will waste gas in case the check did not pass . 

the function `_mintIfThresholdMet()` read from the storage to get the `Transaction` struct corresponding to the `txnHash` and then create a copy on the memory , then checks if the threshold is met or not  , in case of the threshold has not been met yet ,the function will not use the copy of the struct , so it is considered wasting gas . 
```
  function _mintIfThresholdMet(bytes32 txnHash) internal {
    bool thresholdMet = _checkThresholdMet(txnHash);
-->  Transaction memory txn = txnHashToTransaction[txnHash];
 -->  if (thresholdMet) {
      _checkAndUpdateInstantMintLimit(txn.amount);
      if (!ALLOWLIST.isAllowed(txn.sender)) {
        ALLOWLIST.setAccountStatus(
          txn.sender,
          ALLOWLIST.getValidTermIndexes()[0],
          true
        );
      }
      TOKEN.mint(txn.sender, txn.amount);
      delete txnHashToTransaction[txnHash];
      emit BridgeCompleted(txn.sender, txn.amount);
    }
  }
```
so the optimized code should include the reading from storage and creating the copy inside the if condition . 
```diff
  function _mintIfThresholdMet(bytes32 txnHash) internal {
    bool thresholdMet = _checkThresholdMet(txnHash);
-  Transaction memory txn = txnHashToTransaction[txnHash];
    if (thresholdMet) {
+   Transaction memory txn = txnHashToTransaction[txnHash];
      _checkAndUpdateInstantMintLimit(txn.amount);
      if (!ALLOWLIST.isAllowed(txn.sender)) {
        ALLOWLIST.setAccountStatus(
          txn.sender,
          ALLOWLIST.getValidTermIndexes()[0],
          true
        );
      }
      TOKEN.mint(txn.sender, txn.amount);
      delete txnHashToTransaction[txnHash];
      emit BridgeCompleted(txn.sender, txn.amount);
    }
  }
```
another instance : 
the function `setRange()` execute some code,which is reading from storage and creating copy into the memory , before the check ,so including this code after the check will save the gas in case of reversion .
```diff
  function setRange(
    uint256 endTimestamp,
    uint256 dailyInterestRate
  ) external onlyRole(SETTER_ROLE) {
-    Range memory lastRange = ranges[ranges.length - 1];

    // Check that the endTimestamp is greater than the last range's end time
    if (lastRange.end >= endTimestamp) revert InvalidRange();
+    Range memory lastRange = ranges[ranges.length - 1];
    uint256 prevClosePrice = derivePrice(lastRange, lastRange.end - 1);
    ranges.push(
      Range(lastRange.end, endTimestamp, dailyInterestRate, prevClosePrice)
    );
    emit RangeSet(
      ranges.length - 1,
      lastRange.end,
      endTimestamp,
      dailyInterestRate,
      prevClosePrice
    );
  }
```
gas saved : about 200 

**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L337-L353

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L151C1-L171C4

## 3) start the function with the checks will save gas in case of reversion 

consider implement the checks first before the body of the function to save gas in case of revertion , as shown here 
```diff
  constructor(
    address admin,
    address setter,
    address pauser,
    uint256 firstRangeStart,
    uint256 firstRangeEnd,
    uint256 dailyIR,
    uint256 startPrice
  ) {
+   if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
    _grantRole(DEFAULT_ADMIN_ROLE, admin);
    _grantRole(PAUSER_ROLE, pauser);
    _grantRole(SETTER_ROLE, setter);


-    if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
    uint256 trueStart = (startPrice * ONE) / dailyIR;
    ranges.push(Range(firstRangeStart, firstRangeEnd, dailyIR, trueStart));
  }
```

**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L30-L46

# rUSDY contract 

## 1) remove the unneseccary and redundant modifier `whenNotPaused` will save gas . 
the functions `wrap()` and `unwrap()` uses the modifier `whenNotPaused` which exist also in the internal functions `_mintShares()` and `_burnShares()`  that both of the functions `wrap()` and `unwrap()` , 
```
 function _mintShares(address _recipient, uint256 _sharesAmount) internal whenNotPaused returns (uint256) {
```
and the `_burnShares()` 
```
    function _burnShares(address _account, uint256 _sharesAmount) internal whenNotPaused returns (uint256) {

```
one of the two modifier can be removed and it will perform the same functionallity of the modifier , and this the same implementation that used by `transferShares()` which does not use the modifier explicitly but the internal function `_transferShares()` which has the `whenNotPaused` modifier 
so the optimized code will be :
```diff 
-   function wrap(uint256 _USDYAmount) external whenNotPaused {
+   function wrap(uint256 _USDYAmount) external  {

-     function unwrap(uint256 _rUSDYAmount) external whenNotPaused {
+     function unwrap(uint256 _rUSDYAmount) external {

```
**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L434
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L449

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L543-L546

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L575-L578

## 2) unused returned value can be removed in the functions _mintShares() and _burnShares() and save gas .
the functions `_mintShares()` and `_burnShares()` return the `totalShares` as returned value which is not been used when those functions been called and `totalShares` is state variable so it is accessable globaly so there is no need to be returned , and this will waste gas every call to this function . 
```diff
  function _mintShares(
    address _recipient,
    uint256 _sharesAmount
-   ) internal whenNotPaused returns (uint256) {
+   ) internal whenNotPaused  { 

    require(_recipient != address(0), "MINT_TO_THE_ZERO_ADDRESS");


    _beforeTokenTransfer(address(0), _recipient, _sharesAmount);


    totalShares += _sharesAmount;


    shares[_recipient] = shares[_recipient] + _sharesAmount;


-   return totalShares;
```
**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L543-L555

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L575-L601