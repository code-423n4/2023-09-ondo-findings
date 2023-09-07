## Protocol Overview 
This project aims to create cross chain stablecoin backed by real world USD assets deposited in a bank account .
### USDY
USDY is a token that represents real world USD assets deposited in a bank account and earn daily interest , as interest accrues value of USDY increases so we can't consider it a stablecoin . 
### rUSDY
rUSDY is a rebasing token backed by USDY , since USDY price increases and it's not a stablecoin rUSDY holds USDY and as price of USDY increases , rUSDY rebases and adds to it total supply but price still stay stable . 
### RWADynamicRateOracle
RWADynamicRateOracle is the oracle used used by rUSDY to calculate USDY price , as interest rate of bank changes overtime admin of oracle sets time ranges that specifies daily interest rate at different time ranges , so RWADynamicRateOracle calculates price of USDY based on time and daily interest . 
### SourceBridge
SourceBridge and DestinationBridge are implemented to enable cross chain functionality for USDY , it uses axelar to send messages to destination chain , by calling burnAndCallAxelar tokens bridged to destination chain , first tokens are burned , pays the gas and then call contract on destination chain . 
### DestinationBridge
DestinationBridge receives messages from SourceBridge and mint tokens (burned tokens in source) , when _execute function is called it hashes the payload and based on predefined approvals needed for amounts and source chain , attaches a approval threshold to txHash , once needed approvals met , new tokens are minted . 
## Approach taken in evaluating the codebase
After analyzing the whole code base (in scope) and understanding purpose and workflow of project, i created the following checklist of malicious actions and tried to find a way to perform them . 

- transfer or bridge tokens as a blacklisted user . 
- bridge tokens instantly or bypassing approvals needed.
- receive more than burned source chain tokens in destination chain . 
- considered scenarios that bridged tokens get lost . 
- receive more share for wrap USDY in rUSDY contract . 
- approve a tx as non approver .

## Centralization risks
It's worth metion that owners have full control over users assets , they can blacklist users , burn and seize their assets and change custom oracle for USDY price at any time . 
- Owner can burn rUSDY of any user and seize underlying USDY 
- Onwer can freeze users assets by blacklisting them 
## Comment on low issue 
I submitted a low issue indicating that txnToThresholdSet[txnHash] is not deleted after _mintIfThresholdMet, so it can be used if nonce is not spent by another chain , since txnHash calculated by hash of payload , user can bridge using the same nonce and amount from another chain so txnHash will be the same as old tx and approvals can be reused for bridge , however it's possible only if threshold is not set for that amount of tokens because it will overrides old txnToThresholdSet[txnHash] , despite very low likelihood of this issue i think it's better to fix it simply by removing txnToThresholdSet[txnHash] after _mintIfThresholdMet to lower the risks as much as possible . 

### Time spent:
8 hours