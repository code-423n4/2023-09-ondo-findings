# 1. Audit Scope

The Ondo finance project under audit scope consists of tokenomics part; where the rUSDY interest bearing token is managed, and the bridging service where USDY tokens are bridged between approved chains.
The audited codebase consits of appx. 965 nSLoC distributed over 6 contracts.

# 2. Approach Taken

1. first, I gained a high-level understanding of the protocol's objectives and architecture from the provided documentation.
2. then conducted a detailed manual review of the code, trying to identify potential vulnerabilities,and where the code deviates from the intended design (as it's considered a bug).
3. then tried to match-pattern with known vulnerabilities reported in previous interest bearing tokens & bridge service projects.
4. I referred to the tests,changed the test parameters to analyse the what-if scenarios and to write PoCs.
5. finally, I documented my findings with PoCs.

# 3. Codebase Analysis

- The in-scope contracts can be divided into two groups:

#### 1. Bridge Service:

- `SourceBridge` & `DestinationBridge` contracts:
  Where bridging of USDY tokens transaction is initiated in the source bridge (by burning the bridged tokens); and then executed after it gets approved in the source chain (by minting the bridged tokens back to the user); this feature is built on the top of AXELAR service.

#### 2. Tokenomics:

- `rUSDY`,`rUSDYFactory` & `RWADynamicOracle` contracts:
  Where deployment of rUSDY token is managed on different service in `rUSDYFactory`; token management in `rUSDY` and token price settup in `RWADynamicOracle`.

# 4. Centralization Risks

#### `SourceBridge` contract:

1. The owner can drain the contract funds using the `multiexcall` function .

#### `DestinationBridge` contract:

1. The owner can change the mintLimit anytime without restrictions or checked conditions, which is very cruical as this variable determines the number of minted tokens for a specific duration; so any malicious owner can set it to a higher value at some time to privilage some transactions to be executed.
2. The owner can call `rescueTokens` function that's meant to transfer the contract tokns balances to the owner at anytime.
3. The owner can add/remove approvers or change the source contract address of any chain at anytime; by doing this, some transactions might never be executed as their minimum approvals threshold will never be met (see point #2 systematic risk) or as the source contract that has added the transaction became unsupported (see point #1 systematic ris).

#### `DestinRWADynimcOracle` contract:

1. The contract admin can override the changes made by the pauser and setter roles.
2. The admin & setter can set the price range of the USDY token at anytime (not restricted).

#### `rUSDY` contract:

1. The contract admin can change the blockList,allowList & sanctionList at anytime.
2. The bruner role can burn rUSDY tokens from any account at anytime and getting the USDY refunds.

# 5. Systemic Risks

- Some medium/high vulnerabilites were detected:
  1. `DestinationBridge` contract: users will lose their bridged tokens if the approved contract address of the source chain is changed or removed.
  2. `DestinationBridge::setThresholds` : the number of approvers threshold can be greater than the number of availble approvers.
  3. `rUSDY::burn` function: USDY tokens are refunded to the owner address instead of the address that has his tokens burned by the owner.
  4. `DestinationBridge` contract: non approved approver can give the transaction the first approval when he initiates the transaction execution (by invoking `DestinationBridge::execute` function).
  5. Funds in `DestinationBridge` contract can't be rescued if the owner is a blocklisted account in the rescued tokens; so this will leads to funds being stuck and never being retreived.
  6. `DestinationBridge` contract: transactions can be executed even if the contract is paused; which contradicts the purpose of adding this feature in the contract.

# 6. Other Recommendations

1. The approved chains on both the source and destination chains must be ensured to be synchronized; since any call can be initiated by the source contract and then get stuck as this source contract became invalid in the destination chain.

2. A refund mechanism in the source contract must be implemented in order to enable users from getting back their burnt tokens if the transaction on the destination chain has failed due to any reason outside the users control; as not meeting the minimum approvals threshold or the source contract that started the call became invalid in the destination chain.

3. It's Recommended to add an expiration for each transaction alongside with the refund mechanism mentioned above.

4. Checking the number of required approvers required for each amount if it's acheivable with the current available approvers; as the admin can set this threshold to a value greater than the currently available approvers which will result in making the initiated transactions never being executed as the current approvers number will never meet the minimu threshold for that transaction to be executed.

5. It's Recommended to synchronize pausing the `RWADynamicOracle` and `rUSDY` contracts; since pausing the oracle will make the `rUSDY` dysfunctional.

6. The `rUSDY` token price is vulnerable to manipulation: since the price of USDY is set manually by the oracle contract admin/setter; then if the admin/setter accounts got compromised this will lead to manipulate the price; so it's recommended to extract the token price from a chainlink oracle.

# 7. Time Spent

Approximately 18 hours; divided between manually reviewing the codebase, reading documentation, foundry testing, and documenting my findings.


### Time spent:
18 hours