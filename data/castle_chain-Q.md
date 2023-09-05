# destination bridge 
## 1) `rescueTokens()` function should prevent the withdrawal of USDY tokens . 
the function `rescueTokens()`should make sure that the token to be withdrawn is not the core token of the bridge , but any tokens that had been sent by mistake , consider add this check : 
```diff
  function rescueTokens(address _token) external onlyOwner {
+   if(_token == address(TOKEN)) revert();
    uint256 balance = IRWALike(_token).balanceOf(address(this));
    IRWALike(_token).transfer(owner(), balance);
  }
```
**code snippet**
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L322C1-L325C4

## 2) if the numOfApprovals is 0 this will freeze the execute function add prevent receiving the messages . 
in the function `setThresholds()` there is no check if the `numberOfApprovers` is equal to 0 which be able to prevent receiving a messages of this amount . 
consider add a check to make sure that the number of approvers in greater than zero . 
```diff
    for (uint256 i = 0; i < amounts.length; ++i) {
+   if (numOfApprovers[i] == 0 ) revert();
      if (i == 0) {
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      } else {
        if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
          revert ThresholdsNotInAscendingOrder();
        }
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      }
    }
```
**code snippet**
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L255-L279

## 3) consider adding a function to remove a chain from the mapping `chainToApprovedSender` to allow the owner to deactivate receiving messages from a specific chain .
there is only the method to add chain and there is no removing mechanism 
**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L234-L240


## 4) add a check to prevent the approvers to approve a deleted transaction and mint to address(zero) . 
the function `approve()` allow approving a deleted transaction, and if the number reach the threshold the function will mint zero tokens to the zero address , so it is better to prevent this from happen . 
should make sure that if(txn.sender == address(0)) revert();
```
  function approve(bytes32 txnHash) external {
    if (!approvers[msg.sender]) {
      revert NotApprover();
    }
    _approve(txnHash);
    _mintIfThresholdMet(txnHash);
  }
```
**code snippet**
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L197-L203

## 5) using getNumApproved() function instead of getting the length of the array , to incease the modularity and readability of the code 
first need to make the function getNumApproved() public function , then 
```diff
  function _checkThresholdMet(bytes32 txnHash) internal view returns (bool) {
-   TxnThreshold memory t = txnToThresholdSet[txnHash];
-   if (t.numberOfApprovalsNeeded <= t.approvers.length) {
+   if (t.numberOfApprovalsNeeded <= getNumApproved(txnHash)) {
      return true;
    } else {
      return false;
    }
  }
```
**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L177C1-L184C4

## 6) check for zero value , otherwise the minting on the desination chain will stop 
the `_mintLimit` variable should be greater than zero to allow minting tokens on the destination chain , so the check for that should be exist. 

**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L61
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L64-L71
## 7) crucial value `txnHash` should be emitted to allow user to get information about the transaction .
```diff
-  emit MessageReceived(srcChain, srcSender, amt, nonce);
+  emit MessageReceived(srcChain, srcSender, amt, nonce , txnHash );
```
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L113


## 8) typo
 - the name of the mapping `txnToThresholdSet` should be `txnHashToThresholdSet` because it maps from the hash of the transaction to the threshold . 
```diff
-   mapping(bytes32 => TxnThreshold) public txnToThresholdSet;
+   mapping(bytes32 => TxnThreshold) public txnHashToThresholdSet;
```

**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L51

 - the comment say `public functions` but the following function is internal function
```
    /*//////////////////////////////////////////////////////////////
                    Public Functions             
    //////////////////////////////////////////////////////////////*/
```
the comment should be internal function or should move the internal function to another position . 

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L327-L329

# RWADynamicOracle 
## 1) when passing `firstRangeStart` and `firstRangeEnd` in the constructor , the `firstRangeStart` should be equal or less than the current timeStamp , and the `firstRangeEnd` should be greater than the current timeStamp 
this will garantee that the oracle will return the right prices after the deployment immediately  
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

+   if ( firstRangeStart < block.timestamp || firstRangeEnd <= block.timestamp) revert(); 
    _grantRole(DEFAULT_ADMIN_ROLE, admin);
    _grantRole(PAUSER_ROLE, pauser);
    _grantRole(SETTER_ROLE, setter);


    if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
    uint256 trueStart = (startPrice * ONE) / dailyIR;
    ranges.push(Range(firstRangeStart, firstRangeEnd, dailyIR, trueStart));
  }
```

**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L30-L46

# rUSDY contract

## 1) wrong values are emitted can mislead the monitoring mechanism or the frontent or any sytem that relay on the events 
in the contract rUSDY.sol , the function `wrap()` emit two events `Transfer` and `TransferShares` , the event `Transfer` emits the number of rUSDY tokens by calling the function `getRUSDYByShares(_USDYAmount)` which take the shares and returns the amount of tokens , but in this event **the amount of shares is wrong , the correct number of shares is : _USDYAmount * BPS_DENOMINATOR** , also the event `TransferShares` should emit the number of shares but it also emit the wrong value `emit TransferShares(address(0), msg.sender, _USDYAmount);` 

**Recommendations**

consider emitting the right values of shares .
```diff
  function wrap(uint256 _USDYAmount) external whenNotPaused {
    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
    _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
-   emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
-   emit TransferShares(address(0), msg.sender, _USDYAmount);
+   emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR));
+   emit TransferShares(address(0), msg.sender, _USDYAmount * BPS_DENOMINATOR );
```
## 2) allow the user to specify a recipient address
allow the user to specify address to receive the tokens in the function `wrap()` and `unwrap()` . 
```diff
 - function wrap(uint256 _USDYAmount) external whenNotPaused {
 + function wrap(uint256 _USDYAmount , address recipient) external whenNotPaused {
    require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
-   _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
+   _mintShares(recipient, _USDYAmount * BPS_DENOMINATOR);
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
    emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
    emit TransferShares(address(0), msg.sender, _USDYAmount);
```
**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L434-L439
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L449-L456

## 3) allow the user to pass minAmountOut of the tokens in the functions `unwrap()` to protect him from any slippage 
allow user to specify the minimum amount expected to be received after the unwrapping to protect him from slippage 

```diff
-  function unwrap(uint256 _rUSDYAmount) external whenNotPaused {
+  function unwrap(uint256 _rUSDYAmount , uint256 minAmountOut) external whenNotPaused {
    require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
    uint256 usdyAmount = getSharesByRUSDY(_rUSDYAmount);
    if (usdyAmount < BPS_DENOMINATOR) revert UnwrapTooSmall();
    _burnShares(msg.sender, usdyAmount);
+   if ((usdyAmount / BPS_DENOMINATOR) <= minAmountOut ) revert();
    usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
    emit TokensBurnt(msg.sender, _rUSDYAmount);
  }
```
**code snippet**

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L449-L456

## 4) unused returned value can be removed in the functions `_mintShares()` and `_burnShares()` 
the functions `_mintShares()` and `_burnShares()` return the `totalShares` as returned value which is not been used when those functions been called and `totalShares` is state variable so it is accessable globaly so there is no need to be returned 
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
## 5) unused arguments of function `_beforeTokenTransfer()` can be removed . 
```diff
  function _beforeTokenTransfer(
    address from,
    address to,
-   uint256
  ) internal view {
    // Check constraints when `transferFrom` is called to facliitate
    // a transfer between two parties that are not `from` or `to`.
    if (from != msg.sender && to != msg.sender) {
      require(!_isBlocked(msg.sender), "rUSDY: 'sender' address blocked");
      require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");
      require(
        _isAllowed(msg.sender),
        "rUSDY: 'sender' address not on allowlist"
      );
    }
```

**code snippet**
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L629

## 6) avoid shadowing the state variable by change the names of the parameters and and "_" prefix the name 

**code snippet**
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L109-L118