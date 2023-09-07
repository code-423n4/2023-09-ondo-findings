 QA Report

## Low Risk Issues
| Count | Explanation |
|:--:|:-------|
| [L-01] | No checks to confirm that the gas paid by user is enough |  
| [L-02] | In case `DestinationBridge` is paused before `SourceBridge`, User can lose funds |  
| [L-03] | No incentive for approvers to approve can lead to issues in long term |  
| [L-04] | Possible Reorg situation in `rUSDYFactory` |    

| Total Low Risk Issues | 4 |
|:--:|:--:|

### [L-01] No checks to confirm that the gas paid by user is enough

## Proof of Concept

In `SourceBridge` contract, When any user calls `burnAndCallAxelar`:

1. `TOKEN`s are burned from caller.
2. Gas Fees passed by user is paid through `payNativeGasForContractCall`.
3. `callContract` call is made to transfer the token cross chain.

```solidity

  function burnAndCallAxelar( 
    uint256 amount,
    string calldata destinationChain
  ) external payable whenNotPaused {
    string memory destContract = destChainToContractAddr[destinationChain];

    if (bytes(destContract).length == 0) {
      revert DestinationNotSupported();
    }

    if (msg.value == 0) {
      revert GasFeeTooLow();
    }

    TOKEN.burnFrom(msg.sender, amount);

    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

    _payGasAndCallContract(destinationChain, destContract, payload);
  }

  function _payGasAndCallContract(
    string calldata destinationChain,
    string memory destContract,
    bytes memory payload
  ) private {
    GAS_RECEIVER.payNativeGasForContractCall{value: msg.value}(
      address(this),
      destinationChain,
      destContract,
      payload,
      msg.sender
    );

    AXELAR_GATEWAY.callContract(destinationChain, destContract, payload);
  }

```

Here, there is no validation that the Gas Fees paid by user is sufficient to transfer the token cross chain or not.

## Impact

There are transaction won't be successful at the destination chain because of lack of gas fees paid. User might need to manually call 

## Recommended Mitigation Steps

As per Axelar Docs,

>  An application that wants Axelar to automatically execute contract calls on the destination chain needs to do three things:
> 1. Estimate the gasLimit that the contract call will require on your executable contract on the destination chain.
> 2. Call the estimateGasFee method to get the sourceGasFee in the desired gas-payment token on the destination chain.
> 3. Pay the AxelarGasService smart contract on the source chain in the amount calculated in step 2.

Given that the extra cost will be refunded to user, a mapping state variable can be added which can save the estimate gas fees it will cost for different chains which owner can update as well. And then keep a require check to make sure that user are passing enough gas to execute the transaction seamlessly.

### [L-02] In case `DestinationBridge` is paused before `SourceBridge`, User can lose funds

## Proof of Concept

When user wants to let's say transfer `x` amount of tokens from Chain A to Chain B, below is the sequence of events that will occur:

1. User calls `burnAndCallAxelar` on `SourceBridge` contract.
2. Tokens of user will be burnt.
3. `_execute` on destination contract will be called.
4. Once user gets enough approvals, tokens will be minted on destination chain.

Issue here can arise during emergency, if `DestinationBridge` is paused before `SourceBridge`. In this case, user's token will be burnt in the source contract but user will not be able to mint the tokens in the destination contract.

## Impact

Loss of funds for User.

## Recommended

Always makes sure to pause all the source contract first during emergency before destination contracts.


### [L-03] No incentive for approvers to approve can lead to issues in long term 

On destination chain, the last `approver` will be required to pay extra gas fees to perform extra operations of Updating mint limit, minting the tokens to sender etc.

Here, approver has no incentive to do it and gas fees can be quite high in case the chain is ethereum. 

Hence, it is recommended to have an additional function which should be called by user to mint the tokens and perform the operation in `_mintIfThresholdMet` function. Or another option is to introduce a fee on transfer of tokens which will be distributed to approvers to compensate for their gas fees cost.

### [L-04] Possible Reorg situation in `rUSDYFactory`

The `deployrUSDY` function deploys a new `rUSDY` contract using the create.

Some of the chains like Arbitrum and Polygon are suspicious of the reorg attack in case contracts are not deployed with Deployed via `create2` with `salt`.

```solidity
File: rUSDYFactory.sol

  function deployrUSDY(
    address blocklist,
    address allowlist,
    address sanctionsList,
    address usdy,
    address oracle
  ) external onlyGuardian returns (address, address, address) {
    rUSDYImplementation = new rUSDY();
    rUSDYProxyAdmin = new ProxyAdmin();
    rUSDYProxy = new TokenProxy(
      address(rUSDYImplementation),
      address(rUSDYProxyAdmin),
      ""
    );

```

So it is recommended to deploy via `create2` with `salt` only.