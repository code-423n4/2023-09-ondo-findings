### Low risk issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | The admin wont be able to burn rUSDY if the address is blacklisted/sanctioned and not on the allowlist  | 1 | 
| [L&#x2011;02] | When overriding a range the prevRangeClosePrice of the next range is not updated  | 1 |  
| [L&#x2011;03] | If a user wraps a small amount then the calculations will be wrong because of rounding  | 1 | 
| [L&#x2011;04] | If the user bridges more than the max mint limit/max threshold then he will lose everything  | 1 |
| [L&#x2011;05] | lastResetMintTime and currentMintAmount should be reset when the admin sets a new mint limit  | 1 | 
| [L&#x2011;06] | Wrong values are emitted when wrapping | 2 | 
| [L&#x2011;07] | Wrong comment in RWADynamicOracle.sol   | 1 | 
| [L&#x2011;08] | dailyInterestRate can be accidentally set to 0 | 1 |  

Total: 9 instances over 8 issues 

## [L&#x2011;01] The admin wont be able to burn rUSDY if the address is blacklisted/sanctioned and not on the allowlist

The `burn()` function in `rUSDY.sol` allows the admin to seize rUSDY if the user is not legally allowed to own it. It will first burn the shares and then it will transfer the USDY. The problem is that the user can be blacklisted/sanctioned which will make the tx revert because of `_beforeTokenTransfer().` The user also needs to be on the allowlist. 


It is very likely that the user will first be blacklisted to prevent him from transferring his assets before they are seized so the admin wont be able to seize the assets because when burning `_beforeTokenTransfer()` checks if the address is blacklisted/sanctioned or no and reverts.


### Impact

The admin wont be able to seize assets from the user and he will have to unblacklist him and maybe put him on the allowlist to do this which can be a problem because the user can quickly transfer his assets when he is not blacklisted. 




### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L672-L683

```solidity
File: usdy/rUSDY.sol

672: function burn(
673:     address _account,
674:     uint256 _amount
675:   ) external onlyRole(BURNER_ROLE) {
676:     uint256 sharesAmount = getSharesByRUSDY(_amount);
677:
678:     _burnShares(_account, sharesAmount);
```

As you can see here, the admin burn function calls `_burnShares()` which then calls `_beforeTokenTransfer()` which checks if the address is blacklisted/sanctioned and reverts if yes. 



### Recommended Mitigation Steps

Do the exact same thing what `_burnShares()` does but without the `_beforeTokenTransfer()`



## [L&#x2011;02] When overriding a range the prevRangeClosePrice of the next range is not updated

When the admin is overriding a range in `RWADynamicOracle.sol`, the `prevRangeClosePrice` of the next range is not updated so if the admin is overriding any range where the next range is already set, the next range will use the wrong data because the `prevRangeClosePrice` wasnt updated. 


So for example if we have 3 ranges, the first one is being used and the 2 after it are already set, the admin wants to override the second range but when overriding, the `prevRangeClosePrice` of the third range is not updated so if the admin doesnt override the third range too, then it will use the wrong data.




### Impact

The range will use the incorrect data and the price of USDY will be different from what its supposed to be which will then affect rUSDY.


### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L218-L228

```solidity
File: rwaOracles/RWADynamicOracle.sol

218: if (indexToModify == 0) {
219:     uint256 trueStart = (newPrevRangeClosePrice * ONE) / newDailyIR;
220:     ranges[indexToModify] = Range(newStart, newEnd, newDailyIR, trueStart);
221: } else {
222:     ranges[indexToModify] = Range(
223:         newStart,
224:         newEnd,
225:         newDailyIR,
226:         newPrevRangeClosePrice
227:     );
228: }
```

As you can see in overrideRange(), the prevRangeClosePrice of the next range is not updated



### Recommended Mitigation Steps

Consider adding `nextNewPrevRangeClosePrice` which the admin can set for the range that is next.



## [L&#x2011;03] If a user wraps a small amount then the calculations will be wrong because of rounding


If a user wraps a really small amount of USDY then the calculations in balanceOf wont be really accurate because the result will rounded down. 


Example(please not that the units are in wei):

A user wraps `1000000 wei of USDY` and receives 10000000000 shares. 
The price of USDY is `1.01386170 * 1e18` so balanceOf will look like this:

`(10000000000 * 1013861700000000000) / (1e18 * 10000) = 1013861.7` but in Solidity this will get rounded to 1013861 which is not really accurate. 


### Impact

The calculation will round down the result which will not be accurate

### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L434-L440

```solidity
File: usdy/rUSDY.sol

434: function wrap(uint256 _USDYAmount) external whenNotPaused {
435:     require(_USDYAmount > 0, "rUSDY: can't wrap zero USDY tokens");
436:     _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
437:     usdy.transferFrom(msg.sender, address(this), _USDYAmount);
438:     emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
439:     emit TransferShares(address(0), msg.sender, _USDYAmount);
440: }
```

When wrapping, the amount can be as little as 1 wei. 


### Recommended Mitigation Steps

Consider adding a minimum amount check in wrap() to prevent users from depositing too small amounts



## [L&#x2011;04] If the user bridges more than the max mint limit/max threshold then he will lose everything

If the amount being bridged is more than the max mint limit or the max threshold amount then when calling `execute()` on the destination chain the tx will revert because of max mint limit/max threshold checks. 

The problem is that these checks only exist on the destination bridge which will cause the tx on the destination bridge to revert while the tx on the source chain will be executed so the user will lose everything he bridged. 


### Impact
The user will burn all his tokens on the source chain but he wont receive anything on the destination chain because the tx is reverting there.

### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L136-L146

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/MintRateLimiter.sol#L82-L85

```solidity
File: bridge/DestinationBridge.sol

136: if (amount <= t.amount) {
137:     txnToThresholdSet[txnHash] = TxnThreshold(
138:         t.numberOfApprovalsNeeded,
139:         new address[](0)
140:     );
141:     break;
142:   }
143: }
144: if (txnToThresholdSet[txnHash].numberOfApprovalsNeeded == 0) {
145:     revert NoThresholdMatch();
146: }
```

As you can see here, if the amount is bigger than then t.amount then there will be 0 attached thresholds and the tx will revert

```solidity
File: bridge/MintRateLimiter.sol

82: require(
83:     amount <= mintLimit - currentMintAmount,
84:     "RateLimit: Mint exceeds rate limit"
85: );
```

or if the amount is bigger than the mintLimit then this will also revert.

### Recommended Mitigation Steps

Consider checking this on the source chain to prevent users from depositing the max




## [L&#x2011;05] lastResetMintTime and currentMintAmount should be reset when the admin sets a new mint limit

When the admin sets a new mint limit on the destination bridge the `lastResetMintTime` and `currentMintAmount` should be reset so there are no pending txs that can unexpectedly fail because of the new limit. 

### Impact

Some txs can unexpectedly fail and the user will have to execute them again.

### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/MintRateLimiter.sol#L82-L85

```solidity
File: bridge/MintRateLimiter.sol

82: require(
83:     amount <= mintLimit - currentMintAmount,
84:     "RateLimit: Mint exceeds rate limit"
85: );
```

As you can see here, the tx might fail because of the mint limit but if the owner makes the new mint limit smaller and the currentMintAmount/lastResetMintTime are not reset then some txs can unexpectedly fail after its set 


### Recommended Mitigation Steps

When setting the mint limit in setMintLimit(), reset the lastResetMintTime and currentMintAmount so no pending txs fail because of the new limit



## [L&#x2011;06] Wrong values are emitted when wrapping


In `rUSDY.sol::wrap()` wrong values are emitted in the events. 

```solidity
emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
```
`getRUSDYByShares(_USDYAmount)` is wrong here because the user received `_USDYAmount * BPS_DENOMINATOR` amount of shares so the event needs to emit `getRUSDYByShares(_USDYAmount * BPS_DENOMINATOR)`




```solidity
emit TransferShares(address(0), msg.sender, _USDYAmount);
```

This is also wrong because the user received `_USDYAmount * BPS_DENOMINATOR` shares


### Impact

This can lead to confusion and incorrect data being captured by event listeners

### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L438-L439



### Recommended Mitigation Steps

Emit the correct values



## [L&#x2011;07] Wrong comment in RWADynamicOracle.sol

In `RWADynamicOracle.sol:: overrideRange()` there is a wrong comment that says:

```solidity
// Ensure that the newStart time is less than the end time of the previous range
if (newStart < ranges[indexToModify - 1].end) revert InvalidRange();
```

However this should be - "Ensure that the newStart time is not less than the end time of the previous range"


### Impact

Confusing code comments deviates from function logic


### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L211-L212

### Recommended Mitigation Steps

Adjust code comments to follow function logic.




## [L&#x2011;08] dailyInterestRate can be accidentally set to 0 

In `RWADynamicOracle.sol::setRange()` the setter can accidentally set the dailyInterestRate to 0 and then the `getPrice()` will return 0 which could cause big problems

### Impact

`getPrice()` will return 0 and all calculation in rUSDY.sol will return 0 which can cause big problems and make the rUSDY non-functional for some time.

### Code Snippet

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L151-L158

```solidity
File: rwaOracles/RWADynamicOracle.sol

151: function setRange(
152:     uint256 endTimestamp,
153:     uint256 dailyInterestRate
154: ) external onlyRole(SETTER_ROLE) {
155:     Range memory lastRange = ranges[ranges.length - 1];
156:
157:     // Check that the endTimestamp is greater than the last range's end time
158:     if (lastRange.end >= endTimestamp) revert InvalidRange();
```

As you can see there is no check that the dailyInterestRate is 0


### Recommended Mitigation Steps

Add a zero amount check for dailyInterestRate
