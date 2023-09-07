 
# Gas Optimizations Report

This report focuses on Ondo-Finance Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Ondo-Finance protocol are categorised into 07 main areas; with further multiple instances in each of the category.

## Summary
[G-01] Increase number of optimisation runs (01 Instances)
[G-02] Create less memory variables (08 Instances)
[G-03] Revert inside a loop (05 Instances)
[G 04] Using private rather than public for immutable saves gas (06 Instances)
[G 05] abi.encode() is less efficient than abi.encodepacked()(03 Instances)
[G-06] It's cheaper to declare the variable outside the loop (02 Instances)
[G-07] Wastage of deployed gas when return is not present for returns function (06 Instances)


# [G-01] Increase number of optimisation runs (01 Instances)
Optimisation runs are should be changed to 1000 instead of 100. This helps in removing unused opcodes which are the main element of gas consumption.
The gas of contract deployment will be comparably high but will cost less gas when functions are executed multiple times after deployement.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-ondo/blob/main/hardhat.config.ts#L24


# [G-02] Create less memory variables (08 Instances)
Avoid creating lot of memory variables inside a function as they consume more gas. Instead use calldata, as the incoming arguments will be used just during execution.

Link to the Code:
2.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L66
3.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79
4.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L133
5.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L135
6.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L112
7.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L120
8.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L130


# [G-03] Revert inside a loop (05 Instances)
Condition inside for loop will carry on, unless it finds one condition that is not met, it will revert the whole transaction. 
It is recommended that instead of reverting if condition is not true, to just break the loop. This way we don't need to run again the transaction.

Link to the Code: 
1.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L168
2.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L145
3.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L162
4.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L271
5.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L134


# [G 04] Using private rather than public for immutable saves gas (06 Instances)
Private over public saves around 21,000 for immutable in gas during deployment.
Gas consumption also increases with number of variable created with public visibility for both constant and immutable.
Link to the code:
1.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L18
2.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L21
3.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L24
4.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L34
5.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L37
6.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L40


# [G 05] abi.encode() is less efficient than abi.encodepacked()(03 Instances)
Refer to this [link]( https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison).

Link to the Code:
1.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79
2.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L99
3.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L238


# [G-06] It's cheaper to declare the variable outside the loop (02 Instances)
Declaring a variable inside a loop result in variable being redeclared during each loop iteration which consume higher gas.
The variable gets reallocated when declared outside loop making it more gas efficient.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L78
2.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L130


# [G-07] Wastage of deployed gas when return is not present for returns function (06 Instances)

Wastage of gas during deployment; when return is absent for named variable when function returns.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L160
2.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L126
3.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L58
4.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L262
5.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L401
6.	https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L405


