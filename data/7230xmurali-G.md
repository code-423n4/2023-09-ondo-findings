rUSDY.sol contract

1.Updating values should be placed in the unchecked because it Con't Overflow/underflow  to save the gas.

 shares[_recipient] = shares[_recipient] + _sharesAmount;

this can be replaced with 

unchecked{
    shares[_recipient] = shares[_recipient] + _sharesAmount;
}


totalShares += _sharesAmount;
shares[_recipient] = shares[_recipient] + _sharesAmount;

this can be Replaced with 
unchecked{
  totalShares += _sharesAmount;
  shares[_recipient] = shares[_recipient] + _sharesAmount;
}


totalShares -= _sharesAmount;
shares[_account] = accountShares - _sharesAmount;

the Above One be Replaced as
unchecked{
    totalShares -= _sharesAmount;
    shares[_account] = accountShares - _sharesAmount;
}

    *** RWADynamicOracle.sol   contract***
2. second time Calculation of ranges.length causes more gas.
  simulateRange in this function 

  uint256 length = ranges.length;  // Assigned the length.

  Range memory lastRange = ranges[ranges.length - 1];

 the Above line replaces as 
  Range memory lastRange = ranges[length- 1];






