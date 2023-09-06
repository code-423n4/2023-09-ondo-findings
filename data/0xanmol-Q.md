

## [L-1] Should not allow to override the ranges of the past

### Code line

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L186

### Impact

The protocol allows the trusted role to change the override range with all new data. it even allows to change the price of the past ranges which can be problematic in many ways.

1. **Data Integrity**: Changing past ranges could alter historical price data. This could impact any systems or users that rely on this data for decision-making or analysis.

2. **Auditability**: Blockchain systems are often valued for their immutability, which allows for clear audit trails. Changing past data could complicate audits and make it harder to track changes over time.

3. **Predictability**: If past ranges can be changed, it could create uncertainty for users. They might not be able to trust the data they see, as it could be changed in the future.

4. **Smart Contract Interactions**: Other smart contracts might interact with this contract based on its historical data. Changing past ranges could cause unexpected behavior in those contracts, potentially leading to bugs or security vulnerabilities.

These are few of the concerns if past data is changed.


## [L-2] Wrong data passed to ```Tranfer``` event in ```wrap``` function

### Code Line

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L438

### Details

After transferring the USDY from user into the rUSDY contract the Tranfer event is emmited. The event is taking three parameters to,from and amount. inside the emit block the ```getRUSDYByShares``` is called which takes ```_usdyAmount``` as input instead of ```_usdyAmount * BPS_DENOMINATOR```. The function is supposed to calculate and return the total amount of USDY that was deposited but it will return the value with four less zeros.

### Recommendation

Either pass ```_usdyAmount``` directly as a amount or calculate the amount correctly my multipling value by ```BPS_DENOMINATOR```