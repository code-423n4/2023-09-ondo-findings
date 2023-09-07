# Code does not follow the best practice of check-effects-interaction
In transfrerFrom() function _approve function should come before _transfer() function 
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L301-L312

    function transferFrom(
        address _sender,
        address _recipient,
        uint256 _amount
       ) public returns (bool) {
           uint256 currentAllowance = allowances[_sender][msg.sender];
           require(currentAllowance >= _amount, "TRANSFER_AMOUNT_EXCEEDS_ALLOWANCE");

      _transfer(_sender, _recipient, _amount);
      _approve(_sender, msg.sender, currentAllowance - _amount);
      return true;
  }


# Inconsistences in the functions under the comment used to group the functions. 
for example **_mintIfThresholdMet(bytes32 txnHash)** is classified under public function comment whereas the function is internal function.

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L337C3-L337C58
  
     function _mintIfThresholdMet(bytes32 txnHash) internal 

