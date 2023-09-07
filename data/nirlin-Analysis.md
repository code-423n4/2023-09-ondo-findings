##   Description
rUSDY is the rebasing variant of USDY whose supply will be increased or decreased based upon the price returned from the custom oracle set up by the ondo team. 

With this audit ondo also introduced the bridging mechanism to send tokens from one source to another to increase the usability of chain across chains for which ondo utilised the axelar network which have vast network of nodes to help process such cross chain transactions and transfers.


*The key contracts of the protocol for this Audit are*:

- *SourceBridge.sol*: 
    This contract help in sending the tokens to another chain using the axelar gateway `callContract` function

- *DestinationBridge.sol*: 
     This contract is axelar executable contract that exposes the execute function that can only be called by the nodes registered with the axelar and than mints the token to the desired address.

- *MintRateLimiter.sol*: 
     This contract is used by destination bridge to limit the number of tokens minted on the destination per unit of time

- *rUSDYFactory.sol*: 
    This contract is used to deploy the transparent upgradable proxy contracts and implementation contract using factory pattern.

- *rUsDY.sol*: 
     This is the main token contract that is rebasing version of the USDY, user can call the `wrap` and `unwrap` function on this contract to convert their USDY back and forth.



*Existing Patterns*: The protocol uses standard Solidity and Ethereum patterns. It uses the `ERC20` standards for most part (not completely though) and `Accesibility` pattern to grant different roles and also there is `whitelisting` and `pausing` mechanism too.


##  Approach
I took the top to down approach for this audit, so general flow is as follow


1. SourceBridge.sol
2. MintRateLimiter.sol
3. DestinationBridge.sol
4. rUSDY.sol
5. rUSDYFactory.sol

Reviewed the source bridge than destination and limiter side by side. As they were all very connected and certain checks needed to make sure in the flow. rUSDY is highly inspired by sthETH only few changes on how it calculates balance and mint share but for other part it was almost the same contract, which i think is a good approach given the amount of audits Lido did, there is alot of innate security guranteet.

## Codebase Quality
I would say the codebase quality is good but can be improved, there are checks in place to handle different roles, each standard is ensured to be followed. And if something is not fully being followed that have been informed. But still it can be improved in following areas


| Codebase Quality Categories  | Comments |
| --- | --- |
| *Unit Testing*  | Code base is well-tested but there is still place to add fuzz testing and invariant testing and foundry have now solid support for those, I would say best in class.|
| *Code Comments*  | Overall comments in code base were not enough, more commenting can be done to more specifically describe specifically mathematical calculation. At many point if there were more detail commenting and natspec it would have been bit easy for the auditor and less question would have been asked from sponser, saving time for both. |
| *Documentation* | Documentation was to the point, even the known issues were completely described and other wise code was easy to follow along|
| *Organization* | Code orgranization was great for sure. The flow from one contract was perfectly smooth. |

##   Systematic and Centralisation Risks

### 1. Centralization Risk: 

It is important to note that there are some previliged function in the system that if went rogue can cause significant problems for the protocol, for example all the roles being assigned only to the one guradian address on deployment. That is unnecessary there should always be some separation fo concern.
                                        
Than there are rescue functions in destination bridge and one another contract that decrease the credibility overall.

### 2. Systematic Risk
Once of the systematic risk is that protocol is highly dependent upon the axelar network which is not the best cross chain solution. Many cross chain hacks have occurred in past on these bridges. 

Better options are:

1. Layer zero.
2, New chainlink CCIP

Both of them are very well established, and specifically the layer zero have done multiple security reviews and too solid at this point as compared to axelar. Instead use that.



## Conclusion
Overall ondo came up with a very strong solution with some weak points in commenting and less docs and some centralisation risk. But the approach used for the core working of protocol is really solid and up to the industry standard and fixing above recommendation will make it even more robust.


### Time spent:
20 hours

### Time spent:
20 hours