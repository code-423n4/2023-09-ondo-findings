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

# Any user has the ability to invoke the allowance(address _owner, address _spender) function and initialize allowances[_owner][_spender]

Even though no specific amount is approved during this action any user can allowance(address _owner, address _spender) function because the function is public and there is no constraint applied, 

My suggestion is to replace the _owner address with msg.sender as the _owner address. Additionally, it's advisable to refactor the allowance into the approve(address _spender, uint256 _amount) function. This would prevent the creation of unnecessary allowances[_owner][_spender] if the approval process is not successful


https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L256-L261

      function allowance(
         address _owner,
         address _spender
      ) public view returns (uint256) {
        return allowances[_owner][_spender];
      }