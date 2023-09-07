### NC1 - Add interfaces to own folder

[IRWAOracle.sol in /rwaOracles/ folder](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/IRWAOracle.sol) given there is an /interfaces folder in the /contracts folder 

**Impact**-> It affects the code  maintainability, readability, organization, auditability and cleanliness of folders and structure

**Recommendation** -> It is recommended add all interfaces into their own /interface folder for all interfaces or create separate interfaces folders within the contract folders where interface is used
e.g /rwaOracles/interfaces/IRWAOracle.sol

### NC2 - Not using named imports

Imports just get entire files e.g
```solidity
import "contracts/interfaces/IAxelarGateway.sol";
import "contracts/interfaces/IAxelarGasService.sol";
import "contracts/interfaces/IMulticall.sol";
import "contracts/interfaces/IRWALike.sol";
```
vs best and cleaner practise of using named imports
```
import {IAxelarGateway} from "contracts/interfaces/IAxelarGateway.sol";
```

[SourceBridge.sol line 3-6](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L3C1-L6C44)
[DestinationBridge.sol line 18-22,25](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L18)
[rUSDY.sol lines 18-28](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L18)
[rUSDYFactory.sol lines 19-22](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L19)
[RWADynamicOracle.sol lines 18-20](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L18)

**Impact**-> it affects the code organization, readability, interpretability, maintainability, auditability and best practises

**Recommendation** -> It is recommended to import as named imports as in the examples below
```solidity
import {ImportedContractName} from "../ImportedContractFile.sol";
```

### NC3 - Inconsistent use of underscore _ preceding function arguments

It appears for majority contracts all function parameters make use of _ undescore format e.g
SourceBridge.sol lines 40 
```solidity
constructor(
    address _token,
    address _axelarGateway,
    address _gasService,
    address owner
  )
```
As in above expected owner to be _owner but we can see inconsistency. The below are other examples in the code 

[SourceBridge.sol does not use any underscores for all function arguments/parameters](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L3C1-L6C44)

[DestinationBridge.sol does not use any underscores for all function arguments/parameters](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L18)

[rUSDY.sol mixes usage within and amongst functions see example below](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol) 
```
// no underscore for address guardian 
 function __rUSDY_init_unchained(
    address _usdy,
    address guardian,
    address _oracle
  ) internal onlyInitializing {
```

[rUSDYFactory.sol does not use underscores ](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L75)

[RWADynamicOracle.sol does not use underscores](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L104)

**Impact**-> it affects the code interpretability, maintainability, readability and consistency

**Recommendation** -> It is recommended to apply consistent usage of underscore _ to all function arguments

### NC4 - Custom Errors should be prefixed by contract name 

```solidity 
error DestinationNotSupported();
```
Error naming is not indicative of the contract name in which the errors will be thrown. It is best practise for errors to have a prefix of the contract they are applied in.

 [error DestinationNotSupported();](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L189C2-L190C24)
 [error GasFeeTooLow();](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L190C24-L190C24)
[DestinationBridge.sol#L439C3-L448C31](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L439C3-L448C31)
```
  error NotApprover();
  error NoThresholdMatch();
  error ThresholdsNotInAscendingOrder();

  error ChainNotSupported();
  error SourceNotSupported();
  error NonceSpent();
  error AlreadyApproved();
  error InvalidVersion();
  error ArrayLengthMismatch();
```
[rUSDY.sol line 94](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L94)
[RWADynamicOracle.sol lines 334 - 336](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L334C2-L336C23)
```
  error InvalidPrice();
  error InvalidRange();
  error PriceNotSet();
```

**Impact**-> Not naming errors with indication of contract can make error tracking, debuggin, analysis difficult as error data bubbles through chain of calls so it may not be clear if error contract receives is from where

**Recommendation** -> It is recommended the errors be named in accordance to the name of the contract in which they will be thrown e. g error 
```
error SourceBridge__DestinationNotSupported;
```

### NC5 - Use of assembly that is not commented 

Assembly is complex and may not be understood by all developers and auditors. When assembly is used it must be thoroughly commented to explain the code, its dynamics and why used 

[RWADynamicOracle.sol from lines 350...](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/rwaOracles/RWADynamicOracle.sol#L350)
```
assembly {
      switch x
      case 0 {
        switch n
        case 0 {
          z := base
...etc
```

**Impact**-> It affects the code  maintainability, readability, auditability of code 

**Recommendation** -> It is recommended to comment all assembly code parts used 



