#  bridge overview 
## Architecture improvments
## 1) implement a minimum limit to amount of token burnt on the source chain . 
the protocol use a bridge between two chains , source and destination chain .
On the destination chain, there is a limit for the amount of tokens that can be minted over a specified duration, causing the mint function to revert if this limit is exceeded. Conversely, the **source chain imposes no such limit**, allowing users to burn and bridge an unlimited quantity of tokens without reversion. As a result, if a user burns and bridges tokens exceeding the destination chain's minting limit, the call will revert on the destination chain or the message will reaches the destination chain but the user will not get his minted tokens even if the number of approvers exceeds the threshold so the user will not be able to mint his tokens because of the limit on minting amount of tokens . 
**Recommendation**
implement the same limit of minting and burning tokens on the source and destination chains to prevent such an issue . 

## 2) pre-determined quantity of gas 
The primary issue revolves around the lack of predetermined gas limits, which allows users to specify any gas amount, leading to transaction failures and unrecoverable token burns.
 according to [axelar docs](https://docs.axelar.dev/dev/general-message-passing/monitoring) , the way to recover the transaction if it has an insufficient gas issue is to use the UI or the SDK which need the transaction hash to determine which transaction to be recovered , but it will not be a good solution to force the user to moniter the transaction maually , and it is better to predetermine the gas amount to get it from the user 
 **Recommendations**
 1) using external oracle such as [chainLink fast gas oracle](https://data.chain.link/ethereum/mainnet/gas/fast-gas-gwei) which provide the gas price on chain and will allow the protocol to pre-determine the gas cost and take them from the user , which will reduce the risk of insufficient amount of gas error . 
 2) Implementation of Chainlink's CCIP: according to [chainlink CCIP docs](https://docs.chain.link/ccip)
 Chainlink's Cross-Chain Interoperability Protocol (CCIP) brings critical capabilities for seamless communication and interaction between different blockchains , which provide  **Arbitrary Messaging**: This allows for the transmission of custom data to different blockchain smart contracts,  the message transmission will be sufficient to execute the minting tokens proccess on the destination chain . 
 
**The advantages of chainlink CCIP are :**
- Pre-determines the gas needed for transactions, eliminating user-specified gas issues.
- Automatically returns transaction hash (messageId) to users, enabling real-time transaction monitoring. 

## 3) keep tracking of the tokens that had been burnt . 
according to the axelar [docs](https://docs.axelar.dev/learn/network/flow#message-processing-and-relayers:~:text=cross%2Dchain%20protocol.-,Message%20processing%20and%20relayers,-Axelar%20network%20must) , the relayer is not always guaranteed to work ,  
```
These relayer services are a free, operational convenience Axelar provides , anyone can create his own relayer
```
and if the relayer works the bridge can **go down** for any reason , or even the transaction on the destination chain can **revert** due to multiple reasons such as (there is no number of approvers set to this amount so the transaction will revert) , so it will be more efficient to keep track of the burned amount of each user and add mint function  to allow the owner to re-mint the amount of tokens that had been burnt and it supposed to be minted on the destination chain .  

# RWA Dynamic Oracle

## 1) add limits to the start timestamp and the end timestamp of the ranges to avoid unexpected price .
the formula that calculate the price in the oracle use the time as exponential variable and this formula can be represented on this [graph](https://www.desmos.com/calculator/zyq5xecoa1)  , the price increases exponentially over time which mean for one (constant) specified `dailyInterestRate` and `prevClosePrice`  , the slope of the curve varies and increases continuously and the will affect the price and will result in unexpected and inflated prices 


```
currentPrice = (Range.dailyInterestRate ** (Days Elapsed + 1)) * Range.lastSetPrice
```
as you can see in the graph , the price increase exponentially not linear as shown in the [graph](https://github.com/code-423n4/2023-09-ondo/blob/main/screenshot.png?raw=true) in the README of the contest , so if the duration has no limits and the `DaysElapsed` increase over the time , **the price can be inflated and be unexpected due to the exponential increasing .** 
**Recomendations**
1) set limits to the start and the end time of the range. 
2) use this formula instead (use multiplication instead of exponent)
```
currentPrice = (Range.dailyInterestRate * (Days Elapsed + 1)) * Range.lastSetPrice . 
```

# rUSDY 
## add a slippage protection mechanism as the protocols that relay on oracles and external bridges 

the protocol should allow the users to specify the minimun amount of tokens that they receive in order to protect them from slippage that can happen due to alot of issues such as wrong parameters that the admin had set , or one of the bridges on a specific chain goes down or even due to any oracle manipulation , so it will be better to add those slippage protection to ensure better user experience . 
**Recommendations** 
- add the slippage allowance argument in the function `wrap()` and `unWarp()` 

## Time spent 
40 hours 

### Time spent:
40 hours