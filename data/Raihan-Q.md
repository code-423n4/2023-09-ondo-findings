# [L-01] Reentrancy Vulnerability in DestinationBridge._mintIfThresholdMet Function

The function _mintIfThresholdMet(bytes32) in the DestinationBridge contract is vulnerable to a reentrancy attack. This is due to external calls being made before state variables are updated.

Details
The _mintIfThresholdMet(bytes32) function makes external calls to ALLOWLIST.setAccountStatus(txn.sender,ALLOWLIST.getValidTermIndexes()[0],true) and TOKEN.mint(txn.sender,txn.amount). After these calls, the state variable txnHashToTransaction[txnHash] is deleted. This order of operations opens up the potential for a reentrancy attack.

Code Snippet

```solidity
337: function _mintIfThresholdMet(bytes32 txnHash) internal {
338:     bool thresholdMet = _checkThresholdMet(txnHash);
339:     Transaction memory txn = txnHashToTransaction[txnHash];
340:     if (thresholdMet) {
341:         _checkAndUpdateInstantMintLimit(txn.amount);
342:         if (!ALLOWLIST.isAllowed(txn.sender)) {
343:             ALLOWLIST.setAccountStatus(
344:               txn.sender,
345:               ALLOWLIST.getValidTermIndexes()[0],
346:               true
347:             );
348:         }
349:         TOKEN.mint(txn.sender, txn.amount);
350:         delete txnHashToTransaction[txnHash];
351:         emit BridgeCompleted(txn.sender, txn.amount);
352:     }
353: }
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L337-L353

### Recommendation
To mitigate this issue, the check-effects-interactions pattern should be applied. This means that all the state changes should be made before calling external contracts. This can prevent potential reentrancy attacks. The state variable txnHashToTransaction[txnHash] should be deleted before the external calls to ALLOWLIST.setAccountStatus and TOKEN.mint.

# [L-02] Variable Shadowing in DestinationBridge Constructor
The _owner variable in the DestinationBridge constructor shadows the _owner state variable from the Ownable contract, leading to confusion and potential bugs.

Details
In Solidity, shadowing occurs when a variable declared within a certain scope has the same name as a variable declared in an outer scope. In this case, the _owner variable declared in the DestinationBridge constructor has the same name as the _owner state variable declared in the Ownable contract. This can introduce ambiguity and make it unclear which variable is being referred to in different parts of the code.

## Code Snippet:

```solidity
60: constructor(
61:     address _token,
62:     address _axelarGateway,
63:     address _allowlist,
64:     address _ondoApprover,
65:     address _owner,
66:     uint256 _mintLimit,
67:     uint256 _mintDuration
68: )
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L60-L68

# Recommendation
To avoid variable shadowing and enhance code clarity, it is recommended to rename the _owner variable in the DestinationBridge constructor to something else, such as _contractOwner. This will clearly indicate which variable is being referred to in different parts of the code. Here is an example of how the constructor could be updated:

```solidity
60: constructor(
61:     address _token,
62:     address _axelarGateway,
63:     address _allowlist,
64:     address _ondoApprover,
+   65:     address _contractOwner,
66:     uint256 _mintLimit,
67:     uint256 _mintDuration
68: )
```
By making this change, potential confusion and bugs related to variable shadowing can be mitigated.

## [L-03] Weak Pseudo Random Number Generator in RWADynamicOracle._rpow Function
The function _rpow(uint256,uint256,uint256) in RWADynamicOracle.sol uses a weak Pseudo Random Number Generator (PRNG) at line 362. The PRNG is weak because it uses a modulo operation on n, which can be influenced by miners to some extent.

Code Snippet
In RWADynamicOracle.sol:
```
function _rpow(
    uint256 x,
    uint256 n,
    uint256 base
  ) internal pure returns (uint256 z) {
    assembly {
      switch x
      case 0 {
        switch n
        case 0 {
          z := base
        }
        default {
          z := 0
        }
      }
      default {
        switch mod(n, 2) // Weak PRNG at line 362
        case 0 {
          z := base
        }
        default {
          z := x
        }
        ...
    }
  }
```

## Details

In the function _rpow(uint256,uint256,uint256), the modulo operation n % 2 is used to determine the switch case. This operation can be influenced by miners, making it a weak source of randomness. This could potentially lead to predictability in the function's output, which could be exploited by malicious actors.

## Recommendation

Avoid using n % 2 as a source of randomness. Consider using a more secure source of randomness that cannot be influenced by miners. If the randomness is not critical for the security of the contract, consider documenting this fact clearly in the code. If the randomness is critical, consider using a secure source of randomness such as the VRF (Verifiable Random Function) provided by Chainlink.

## [L-04] Unchecked Return Value in rUSDY.burn Function Posing Potential Risk
The rUSDY.burn function does not check the return value of the usdy.transfer function call. This could potentially lead to unexpected behavior if the transfer fails but the contract continues to execute.

Location
The issue is located in the burn function of the `rUSDY` contract

```solidity
  function burn(
    address _account,
    uint256 _amount
  ) external onlyRole(BURNER_ROLE) {
    uint256 sharesAmount = getSharesByRUSDY(_amount);

    _burnShares(_account, sharesAmount);

    usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);

    emit TokensBurnt(_account, _amount);
  }
```

# Description
In the burn function, the usdy.transfer function is called to transfer the burned shares (USDY) to msg.sender. However, the return value of this function call is not checked. This means that if the transfer fails for any reason, the contract will not be aware of this and will continue to execute. This could potentially lead to unexpected behavior.

# Recommendation
To mitigate this issue, it is recommended to use the SafeERC20 library provided by OpenZeppelin, which automatically checks the return value of transfer and transferFrom calls. If the SafeERC20 library cannot be used, then it is recommended to manually check the return value of the transfer function call to ensure that it was successful. If the transfer was not successful, the contract should revert the transaction to prevent any state changes