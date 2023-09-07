https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

 _mintIfThresholdMet function

1)You can save gas by using a modifier or function guard to check whether txnHash exists in txnHashToTransaction before proceeding with the operation. This avoids the cost of accessing a non-existent entry.
modifier transactionExists(bytes32 txnHash) {
    require(txnHashToTransaction[txnHash].amount > 0, "Transaction does not exist");
    _;
}

function _mintIfThresholdMet(bytes32 txnHash) internal transactionExists(txnHash) {
    bool thresholdMet = _checkThresholdMet(txnHash);
    Transaction memory txn = txnHashToTransaction[txnHash];
    if (thresholdMet) {
        _checkAndUpdateInstantMintLimit(txn.amount);
        if (!ALLOWLIST.isAllowed(txn.sender)) {
            ALLOWLIST.setAccountStatus(
                txn.sender,
                ALLOWLIST.getValidTermIndexes()[0],
                true
            );
        }
        TOKEN.mint(txn.sender, txn.amount);

        // Delete the transaction entry after processing
        delete txnHashToTransaction[txnHash];

        emit BridgeCompleted(txn.sender, txn.amount);
    }
}
We introduced a modifier called transactionExists that checks whether the transaction with the given txnHash exists in txnHashToTransaction. If the transaction doesn't exist (i.e., the amount is zero or less), it will revert the function call.
Inside the _mintIfThresholdMet function, we use the transactionExists modifier to ensure that the transaction exists before proceeding with the logic.
After processing the transaction, we delete the entry in txnHashToTransaction as in your original code.
This way, we avoid the cost of accessing a non-existent entry in txnHashToTransaction by using the transactionExists modifier to check for its existence before executing the main logic.
2)Instead of deleting the transaction entry in txnHashToTransaction (delete txnHashToTransaction[txnHash]), you can consider using a flag to mark it as processed and leave the entry. This way, you avoid the cost of deleting and save on gas.
// Define a mapping to keep track of processed transactions
mapping(bytes32 => bool) public processedTransactions;

function _mintIfThresholdMet(bytes32 txnHash) internal {
    bool thresholdMet = _checkThresholdMet(txnHash);
    Transaction memory txn = txnHashToTransaction[txnHash];
    if (thresholdMet) {
        _checkAndUpdateInstantMintLimit(txn.amount);
        if (!ALLOWLIST.isAllowed(txn.sender)) {
            ALLOWLIST.setAccountStatus(
                txn.sender,
                ALLOWLIST.getValidTermIndexes()[0],
                true
            );
        }
        TOKEN.mint(txn.sender, txn.amount);

        // Mark the transaction as processed using a flag
        processedTransactions[txnHash] = true;

        emit BridgeCompleted(txn.sender, txn.amount);
    }
}
We introduced a mapping(bytes32 => bool) public processedTransactions to keep track of processed transactions. Each transaction hash serves as a key, and the boolean value indicates whether the transaction has been processed.
Inside the _mintIfThresholdMet function, after processing the transaction's logic, we mark the transaction as processed by setting processedTransactions[txnHash] to true. This flag indicates that the transaction has been processed.
With this approach, you avoid the gas cost associated with deleting entries in txnHashToTransaction. Instead, you simply set a flag in the processedTransactions mapping, making the code more gas-efficient.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

simulateRange function

1)In this function, consider using a storage variable to store the ranges array instead of repeatedly accessing it from storage, which can be gas expensive if ranges is a large array.
Range[] storage rangeList = ranges;
2)Redundant Range Copying: The code creates a copy of the ranges array into rangeList, which could be avoided by directly working with ranges. This would save gas and memory.

overrideRange function
1)Gas Efficiency: Consider breaking down the checks into separate functions. This can help improve gas efficiency by allowing the contract to exit early when a condition fails. It also makes the code more readable and maintainable.

getPrice function
1)Consider using a while loop instead of a for loop. This way, you can exit the loop as soon as you find the appropriate range. This could be more gas-efficient if there are many ranges, and the desired range is near the beginning.


