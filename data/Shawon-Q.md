1.`setMintLimit` and `setMintLimitDuration` function of DestinationBridge.sol has no input value validation,which by accident if set to ,the users may have to face
uninted situations.Setting an input validation is recommended

2.safe transfer from openzeppelin is missing for transfering tokens in rusdy.sol

3.There are several functions inside the rUSDY.sol which has requirements such as:
 `- the contract must not be paused.`
but we see no such implementations of these requirements on the functions related to the comments.
Dissimilarities between functions and its requirements are signs of logical error.

4.The _setOracle function of rUSDY.sol should emit an event to monitor off-chain to improve transparancy in case the oracle changes;
as it does critical state changes and can change the overall behaviors of the contract
