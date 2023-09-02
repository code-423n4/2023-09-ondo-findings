**contracts/bridge/SourceBridge.sol**
- L46/47/48 - Validation of address != 0x is missing in the constructor, since they are immutable variables.


**contracts/bridge/DestinationBridge.sol**
- L67/68/69 - Validation of address != 0x is missing in the constructor, since they are immutable variables.

- L179/180/181/182/183 - You could simplify the if else into: "return t.numberOfApprovalsNeeded <= t.approvers.length;"


**contracts/usdy/rUSDYFactory.sol**
- L46 - There is no validation of address != 0x, in the constructor, since they are immutable variables.


**contracts/rwaOracles/RWADynamicOracle.sol**
- L44/117/219 - A division is made by the input, dailyIR, therefore if a 0 is put, it could generate an exception, for that reason we must add a previous validation.


**contracts/rwaOracles/IRWADynamicOracle.sol**
- This interface is only used in the rUSDY contract, therefore, instead of creating a file, you could directly put the interface inside the rUSDY.sol file
