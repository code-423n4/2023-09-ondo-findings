## rUSDY.sol
@line493, should check if owner actually owns the amount
@line 485 approve should check that msg.sender is owner 
@line 310 approve return value is not checked
@line 334 added value should be greater than or equal 0 
@line 353 method allows the input of negative numbers
## sourcebridge.sol
@line 72 should have a min vale as a constant, as values like 0.0000001 is allowed