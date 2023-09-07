## 1. `txnToThresholdSet[txnHash]` RECORD HAS NOT BEEN DELETED AFTER THE TRANSACTION HAS BEEN MINTED

The `DestinationBridge._mintIfThresholdMet` function is used to mint a transaction if it has passed the threshold for the number of approvers. If the particular transaction with the provided `txnHash` has fulfilled the required approval threshold then the respective transaction will be minted and the record for the particular `txnHash` in the `txnHashToTransaction` mapping will be deleted as shown below:

      TOKEN.mint(txn.sender, txn.amount);
      delete txnHashToTransaction[txnHash];

But the issue here is that the `txnToThresholdSet[txnHash]` record for this particular `txnHash` which was minted, is not deleted.

Hence it is recommended to delete the `txnToThresholdSet[txnHash]` record in the `DestinationBridge._mintIfThresholdMet` function after the transactino has been minted as shown below:

      delete txnHashToTransaction[txnHash]; 
      delete txnToThresholdSet[txnHash];       

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L350

## 2. IT IS RECOMMENED TO FOLLOW THE `CEI` PROCESS WHEN CALLING EXTERNAL FUNCTIONS TO TRANSFER FUNDS

The `rUSDY.wrap` function is used by the users to wrap thier `USDY` into `rUSDY`. The function mint shares to the `msg.sender` and receives the `USDY` tokens from the msg.sender as shown below:

    _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);

But it does not follow the `CEI` flow. Hence it is recommended to mint the shares to the `msg.sender` after the `USDY` has been transferred from the `msg.sender` to the `rUSDY` contract.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L436-L437

## 3. THE CRITICAL FUNCTION STATE CHANGES DO NOT HAVE EVENTS EMITTED WITH OLD AND NEW VALUES

The `rUSDY.setOracle`, `rUSDY.setBlocklist`, `rUSDY.setAllowlist` and `rUSDY.setSanctionsList` are called by the `LIST_CONFIGURER_ROLE` user to set critical addresses of the procotol. But none of these functions emit events. 

Hence it is recommended to `emit events` for these critical functions by including `both old and newly updated values`, for future reference (logging).

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L662-664
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L698-L702
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L709-L713
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L720-L724

## 4. STATE VARIABLES ARE SHADOWED BY INPUT VARIABLE NAMES IN CRITICAL FUNCTIONS

The input variables to critical functoins such as `rUSDY.setBlocklist`, `rUSDY.setAllowlist` and `rUSDY.setSanctionsList` are overshadowing the state varaibles of inherited contracts. Hence it is recommended to change the names of input variables of above functions to avoid this issue.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L701
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L712
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L723

## 5. LACK OF `address(0)` CHECK IN THE `DestinationBridge.addApprover` FUNCTION

The `DestinationBridge.addApprover` function is used to add an ondo Signer or Axelar Relayer to the `approvers` mapping. But there is no input validation for the passed in `address approver` input variable.

Hence it is recommended to perform the address(0) check on the `address approver` variable in the `DestinationBridge.addApprover` function.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L210-L213

## 6. `RWADynamicOracle.getPrice` FUNCTION CAN BE MODIFIED TO INCREASE ITS EFFICIENCY DURING EXECUTION

The `RWADynamicOracle.getPrice` function is used to return the daily price of USDY given the range previously set. The `getPrice` function uses a `for` loop to iterate through the `ranges` array to derive the price based on the current `block.timestamp` as shown below:

    for (uint256 i = 0; i < length; ++i) {
      Range storage range = ranges[(length - 1) - i];
      if (range.start <= block.timestamp) {
        if (range.end <= block.timestamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, block.timestamp);
        }
      }
    }

Above `for` loop can be modified as follows to increase the efficiency of its execution, by reducing the number of times `if` conditional statements are executed.

    for (uint256 i = 0; i < length; ++i) {
      Range storage range = ranges[(length - 1) - i];
      if (range.end <= block.timestamp) {
        return derivePrice(range, range.end - 1);
      } else if (range.start <= block.timestamp) {
        return derivePrice(range, block.timestamp);        
      }
    }

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L77-L86