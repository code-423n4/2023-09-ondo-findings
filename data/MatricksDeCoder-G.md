### GAS-1 abi.encode is less efficient than abi.encodePacked

abi.encodePacked packs more data tightly in byte array saving more space, it also allows usage of data in more efficient way, abi.encodePacked by avoiding padding uses minimal memory required to encode data all these lead to gas savings 

[...keccak256(abi.encode(srcAddr))).  DestinationBridge.sol line 99](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L99)

[...keccak256(abi.encode(srcContractAddress)).  DestinationBridge.sol line 238](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L238)

### GAS-2 unused imports 

unused imports increase contract bytecode for no benefit leading to bigger contract size that affects deployment costs

[import "contracts/interfaces/IAxelarGasService.sol";](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L19C30-L19C47)

### GAS-3 do while loops are cheaper than for loops

do while loops are cheaper than for loops used in the contracts

[DestinationBridge.sol line 134](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L134)

[DestinationBridge.sol line 160](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L160)

[DestinationBridge.sol line 264](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L264)

[USDYFactory.sol line 130](https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L130)

