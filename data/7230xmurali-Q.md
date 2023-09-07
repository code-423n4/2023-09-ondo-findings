**Low Issues**
rUSDY.sol Contract.

1. In the _transferShares function check the bool value.

function _transferShares(
    address _sender,
    address _recipient,
    uint256 _sharesAmount
  ) internal whenNotPaused {
    
    require(_sender != address(0), "TRANSFER_FROM_THE_ZERO_ADDRESS");
    require(_recipient != address(0), "TRANSFER_TO_THE_ZERO_ADDRESS");

    _beforeTokenTransfer(_sender, _recipient, _sharesAmount);

    uint256 currentSenderShares = shares[_sender];
    require(
      _sharesAmount <= currentSenderShares,
      "TRANSFER_AMOUNT_EXCEEDS_BALANCE"
    );
    //@audit unchecked.
    shares[_sender] = currentSenderShares - _sharesAmount;
    shares[_recipient] = shares[_recipient] + _sharesAmount;
  }
this code will be Replaced with

function _transferShares(
    address _sender,
    address _recipient,
    uint256 _sharesAmount
  ) internal whenNotPaused  returns(bool){
    
    require(_sender != address(0), "TRANSFER_FROM_THE_ZERO_ADDRESS");
    require(_recipient != address(0), "TRANSFER_TO_THE_ZERO_ADDRESS");

    _beforeTokenTransfer(_sender, _recipient, _sharesAmount);

    uint256 currentSenderShares = shares[_sender];
    require(
      _sharesAmount <= currentSenderShares,
      "TRANSFER_AMOUNT_EXCEEDS_BALANCE"
    );
    //@audit unchecked.
    shares[_sender] = currentSenderShares - _sharesAmount;
    shares[_recipient] = shares[_recipient] + _sharesAmount;
    
   ++return true.
  }
and check that value in the where _transferShares function used.

function _transfer(
    address _sender,
    address _recipient,
    uint256 _amount
  ) internal {
    uint256 _sharesToTransfer = getSharesByRUSDY(_amount);
    ++(bool success)=_transferShares(_sender, _recipient, _sharesToTransfer);
    ++if(!success){
       revert ("failed transfer");
      }
    emit Transfer(_sender, _recipient, _amount);
    emit TransferShares(_sender, _recipient, _sharesToTransfer);
  }

this will be helpful when the transfer fail they show that transaction failed.

*** implement this one in all functions which uses the _transferShares function.


****Non Critical Issues***
1.approve Amount must and should !=0.

function _approve(
    address _owner,
    address _spender,
    uint256 _amount
  ) internal whenNotPaused {
    //@audit _amount >0.
    require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");
    require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");

    allowances[_owner][_spender] = _amount;
    emit Approval(_owner, _spender, _amount);
  }

this can be replaced as 

function _approve(
    address _owner,
    address _spender,
    uint256 _amount
  ) internal whenNotPaused {
    //@audit _amount >0.
   ++ if (_amount ==0){
       revert ("amount must >0");
    require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");
    require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");

    allowances[_owner][_spender] = _amount;
    emit Approval(_owner, _spender, _amount);
  }

 And also check the  increaseAllowance  function _addedValue Also !=0.



