|   No.   |   Issue  | Instances |
|---------|----------|-----------|
|[G-01]| `++nonce` costs less gas than `nonce++` |1|
|[G-02]| Deploying `private` state variables costs less gas than deploying `public` state variables | 7 |



## [G-01] `++nonce` costs less gas than `nonce++`

### Instances

[SourceBridge.sol/#L79](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79)
```solidity
bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
```

### Issue

`++i` (pre-increment) costs less gas than `i++` (post-increment). The reason is that `i++` increments the value of `i` but returns the original value. This means the EVM needs to temporarily store the original value before incrementing `i`. In the case of `++i`, however, the EVM can increment the value of `i` right away and simply return the incremented value; no need for temporary storage.

So, roughly speaking, the EVM will operate as follows:

`i++`:
1. Store the current value of `i` temporarily.
2. Increment `i`.
3. Return the stored (original) value.

`++i`:
1. Increment `i`.
2. Return the new value of `i`.

In particular, this means that pre- and post-increment are generally *not* equivalent operations. For example, doing

```solidity
i = 0;
array[i++];
```

will increment `i` by 1 but access the array at index 0. On the contrary, using `array[++i]` will increment `i` by 1, too, but access the array at index 1.

So, depending on the context, one *cannot* always replace `i++` with `++i` to save gas without changing the contract's logic.

### Recommendation

In the context of this project, the post-increment is exclusively used for the `nonce` whose only purpose is to ensure a unique `payload`. In particular, whether the `nonce`s start at 0 (using `nonce++`) or at 1 (using `++nonce`) isn't relevant. In either case, the `payload` is going to be unique. For this reason, I suggest the following change:

[SourceBridge.sol/#L79](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79)
```diff
-    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
+    bytes memory payload = abi.encode(VERSION, msg.sender, amount, ++nonce);
```

## [G-02] Deploying `private` state variables costs less gas than deploying `public` state variables

### Instances

[SourceBridge.sol/#L30](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L30)
```solidity
uint256 public nonce;
```

[rUSDY.sol/#L85](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol/#L85)
```solidity
IRWADynamicOracle public oracle;
```

[rUSDY.sol/#L88](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol/#L88)
```solidity
IUSDY public usdy;
```

[DestinationBridge.sol/#L43-L45](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol/#L43-L45)
```solidity
mapping(address => bool) public approvers;
mapping(string => bytes32) public chainToApprovedSender;
mapping(bytes32 => mapping(uint256 => bool)) public isSpentNonce;
```

[DestinationBridge.sol/#L51-L53](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol/#L51-L53)
```solidity
mapping(address => bool) public approvers;
mapping(string => bytes32) public chainToApprovedSender;
mapping(bytes32 => mapping(uint256 => bool)) public isSpentNonce;
```

[rUSDYFactory.sol/#L47-L49](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol/#L47-L49)
```solidity
rUSDY public rUSDYImplementation;
ProxyAdmin public rUSDYProxyAdmin;
TokenProxy public rUSDYProxy;
```

[RWADynamicOracle.sol/#L25](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol/#L25)
```solidity
Range[] public ranges;
```

### Issue

While the gas costs of reading a state variable from storage inside a contract function don't change based on its visibility, the state variable's visibility *does* make a difference in gas costs during deployment. To convince ourselves that this is indeed the case, we can deploy the following two PoC contracts in [Remix](https://remix.ethereum.org/) and compare the deployment costs:

#### Contract 1

```solidity
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

contract Test {
    uint256 public myVar;
}
```

#### Contract 2

```solidity
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

contract Test {
    uint256 private myVar;
}
```

| |   Gas   |   Transaction Cost  | Execution Cost |
|-|---------|---------------------|----------------|
| **Contract 1** | 105891 | 92093 | 35887 |
| **Contract 2** | 77126 | 67072 | 12666 |

#### Additional Note

Besides being less gas-efficient during deployment, the `public` visibility suggests that the variable's value has some usage *outside* the contract. So, whenever this is not the case, choosing `public` visibility doesn't properly reflect the variable's intended usage. This makes the code harder to understand for both auditors and other team members.

### Recommendation

The following suggestions assume that no other services outside the provided codebase (e.g., a front-end) must call the variables' implicit getter functions.

If this *should* be the case for some of the findings, I still recommend marking the variables as `private` and implementing explicit getter functions instead. This approach doesn't make a difference in gas costs but makes the intended usage of the variables much more transparent. In my humble opinion, implementing explicit getters is generally the better practice because the implicit approach can sometimes introduce subtle and unexpected inheritance behavior. Consider, for example, the following code:

```solidity
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

contract Risky {
    string public vital;

    constructor() {
        vital = "do not change";
    } 
}

contract Careless is Risky {
    function oops() public {
        /*
        Since vital is public, this child contract is free to write to it.
        This is not ideal because it could lead to critical values being overwritten.
        This child contract is free to make mistakes that could be disastrous.
        */

        vital = "game over";
    } 
}

/*
------------------------
Using private for safety
------------------------
*/

contract Cautious {
    string private vital;

    constructor() {
        vital = "cannot be changed by accident";
    }

    function getVital() public view returns(string memory) {
        return vital;
    }

    function setVital(string memory youKnowWhatYoureDoing) public {
        vital = youKnowWhatYoureDoing;
    }
}

contract Isolated is Cautious {
    /*
    This child cannot change Cautious' vital by accident because vital's visibility is neither public nor internal.
    It *must* use Cautious' getter and setter functions. Thus, Cautious is in full control.
    */
} 
```

With all of that out of the way, here are my recommendations:

[SourceBridge.sol/#L30](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L30)
```diff
-    uint256 public nonce;
+    uint256 private nonce;
```

[rUSDY.sol/#L85](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol/#L85)
```diff
-    IRWADynamicOracle public oracle;
+    IRWADynamicOracle private oracle;
```

[rUSDY.sol/#L88](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol/#L88)
```diff
-    IUSDY public usdy;
+    IUSDY private usdy;
```

[DestinationBridge.sol/#L43-L45](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol/#L43-L45)
```diff
-    mapping(address => bool) public approvers;
-    mapping(string => bytes32) public chainToApprovedSender;
-    mapping(bytes32 => mapping(uint256 => bool)) public isSpentNonce;
+    mapping(address => bool) private approvers;
+    mapping(string => bytes32) private chainToApprovedSender;
+    mapping(bytes32 => mapping(uint256 => bool)) private isSpentNonce;
```

[DestinationBridge.sol/#L51-L53](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol/#L51-L53)
```diff
-    mapping(address => bool) public approvers;
-    mapping(string => bytes32) public chainToApprovedSender;
-    mapping(bytes32 => mapping(uint256 => bool)) public isSpentNonce;
+    mapping(address => bool) private approvers;
+    mapping(string => bytes32) private chainToApprovedSender;
+    mapping(bytes32 => mapping(uint256 => bool)) private isSpentNonce;
```

[rUSDYFactory.sol/#L47-L49](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol/#L47-L49)
```diff
-    rUSDY public rUSDYImplementation;
-    ProxyAdmin public rUSDYProxyAdmin;
-    TokenProxy public rUSDYProxy;
+    rUSDY private rUSDYImplementation;
+    ProxyAdmin private rUSDYProxyAdmin;
+    TokenProxy private rUSDYProxy;
```

[RWADynamicOracle.sol/#L25](https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol/#L25)
```diff
-    Range[] public ranges;
+    Range[] private ranges;
```