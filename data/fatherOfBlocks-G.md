**contracts/bridge/SourceBridge.sol**
- L72 - You could first validate that msg.value == 0, in the burnAndCallAxelar function, since if it is reversed, the extra gas would not be used to create the variable and so on.


**contracts/usdy/rUSDY.sol**
- L310/362/530 - It can be unchecked, since a previous validation is carried out.


**contracts/rwaOracles/RWADynamicOracle.sol**
- L334/336 - InvalidPrice and PriceNotSet errors are created, but they are never used, therefore they should be eliminated.
