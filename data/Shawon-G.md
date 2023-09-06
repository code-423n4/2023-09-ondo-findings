Inside function `burnAndCallAxelar` of SourceBridge.sol there are input parameters  `uint256 amount`,
`string calldata destinationChain`.These input parameters are checked inside the function.
 If we check before entering the function, we can save gas.
We can check them using require statements and if the require statement evaluates to false, 
then the function will revert and the gas used to enter the function will be refunded. 
This can help to save gas in cases where the parameters are invalid.