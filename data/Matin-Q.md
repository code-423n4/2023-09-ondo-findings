### Low-Risk Issues

|  | Issue | Instances |
| --- | --- | --- |
| [L‑01] | `indexed` keyword for reference type variables such as `string` in events may lead to data loss. | 1 |

Total: 1 instance over 1 issue

Note: The above tables were created, considering the automatic findings and thus, those are not included.

---

## Low-Risk Issues

### [L‑01] **`indexed` keyword for reference type variables such as `string` in events may lead to data loss**

Summary:

when the ```indexed``` keyword is used for reference typed variables such as string, it will return the hash of the mentioned string.
Thus, the event which is supposed to inform all of the applications subscribed to its emitting transaction (e.g. front-end of the DApp), would get a meaningless and obscure 32 bytes that correspond to keccak256 of an encoded string. For more information about the `indexed` events, one can [check here](https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=indexed#events).

*There are 2 instances of this issue:*

```solidity
  event DestinationChainContractAddressSet(
    string indexed destinationChain,
    address contractAddress
  );
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L183C1-L186C5
