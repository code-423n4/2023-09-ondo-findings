## [L-01] Block values as a proxy for time
Description
Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners can't set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers can't rely on the preciseness of the provided timestamp.

```txt
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L64
    timestamp = block.timestamp;

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L79-L83
      if (range.start <= block.timestamp) {
        if (range.end <= block.timestamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, block.timestamp);
```

Remediation
Developers should write smart contracts with the notion that block values are not precise, and the use of them can lead to unexpected effects. Alternatively, they may make use oracles.

## [L-02] Unsafe erc20 operations
Description
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.
```txt
2023-09-ondo/contracts/bridge/DestinationBridge.sol::324 => IRWALike(_token).transfer(owner(), balance);
2023-09-ondo/contracts/usdy/rUSDY.sol::437 => usdy.transferFrom(msg.sender, address(this), _USDYAmount);
2023-09-ondo/contracts/usdy/rUSDY.sol::454 => usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
2023-09-ondo/contracts/usdy/rUSDY.sol::680 => usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);
```