(1) :- `transferShares` implemented but `transferSharesFrom` isn't like lido's stEth in rUSDY.sol https://etherscan.io/address/0x17144556fd3424edc8fc8a4c940b2d04936d17eb#code#F27#L353
-> this would make $rUSDY's shares easy to use in dapps in the future, that's why there's is a _transferShares() internal function in the first place 
```solidity
    function transferSharesFrom(
        address _sender, address _recipient, uint256 _sharesAmount
    ) external returns (bool) {
        uint256 currentAllowance = allowances[_sender][msg.sender];
        require(currentAllowance >= _amount, "TRANSFER_AMOUNT_EXCEEDS_ALLOWANCE");
        _transferShares(_sender, _recipient, _sharesAmount);
        emit TransferShares(_sender, _recipient, _sharesAmount);
        uint256 tokensAmount = getRUSDYByShares(_sharesAmount);
        emit Transfer(msg.sender, _recipient, tokensAmount);
        _approve(_sender, msg.sender, currentAllowance - _amount);
        return true;
    }
```