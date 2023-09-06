## With the right conditions there might be some dust left in the contract that won't be able to be swept 
Since unwrapping the shares you get back is based on the price there can be a price or constraint that leaves `9999`wei of tokens stuck in the contract.
ex:
Alice has wrapped 1 token 
1 day later Alice wants to unwrap it token. so she gets back `9999999999999999999000`
which means there are 100 wei stuck in the contract. She shouldn't need to wrap more tokens to get it back.

## Since the system works that when there is a smaller price more shares are required
So when there is depegg or short drop in price users make TXs after a price update and won't be able to withdraw
So if the amount users want to  unwrap is this whole balance then when there is a depeg 
There are more shares causing a revert since this balance cant clear that many shares