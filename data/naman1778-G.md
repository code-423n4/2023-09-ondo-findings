## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

There are 3 instances of this issue in 2 files:

```
File: contracts/bridge/SourceBridge.sol

79: bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
```

    diff --git a/contracts/bridge/SourceBridge.sol b/contracts/bridge/SourceBridge.sol
    index 457bf41..637934b 100644
    --- a/contracts/bridge/SourceBridge.sol
    +++ b/contracts/bridge/SourceBridge.sol
    @@ -76,7 +76,7 @@ contract SourceBridge is Ownable, Pausable, IMulticall {
         // burn amount
         TOKEN.burnFrom(msg.sender, amount);

    -    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);
    +    bytes memory payload = abi.encodePacked(VERSION, msg.sender, amount, nonce++);

         _payGasAndCallContract(destinationChain, destContract, payload);
       }

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol

```
File: contracts/bridge/DestinationBridge.sol

99: if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {

238: chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));
```

    diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
    index 8ad410c..b79d28e 100644
    --- a/contracts/bridge/DestinationBridge.sol
    +++ b/contracts/bridge/DestinationBridge.sol
    @@ -96,7 +96,7 @@ contract DestinationBridge is
         if (chainToApprovedSender[srcChain] == bytes32(0)) {
           revert ChainNotSupported();
         }
    -    if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {
    +    if (chainToApprovedSender[srcChain] != keccak256(abi.encodePacked(srcAddr))) {
           revert SourceNotSupported();
         }
         if (isSpentNonce[chainToApprovedSender[srcChain]][nonce]) {
    @@ -235,7 +235,7 @@ contract DestinationBridge is
         string calldata srcChain,
         string calldata srcContractAddress
       ) external onlyOwner {
    -    chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));
    +    chainToApprovedSender[srcChain] = keccak256(abi.encodePacked(srcContractAddress));
         emit ChainIdSupported(srcChain, srcContractAddress);
       }

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There is 1 instance of this issue in 1 file:

```
File: contracts/rwaOracles/RWADynamicOracle.sol

405: require(y == 0 || (z = x * y) / y == x);
```

    diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
    index 03aa7d6..9fcce67 100644
    --- a/contracts/rwaOracles/RWADynamicOracle.sol
    +++ b/contracts/rwaOracles/RWADynamicOracle.sol
    @@ -402,6 +402,6 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
       }

       function _mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
    -    require(y == 0 || (z = x * y) / y == x);
    +    require((y ^ 0) & (((z = x * y) / y) ^ x));
       }
     }

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }

    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }

    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-03] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

There are 3 instances of this issue in 1 file:

```
File: contracts/rwaOracles/RWADynamicOracle.sol

117: uint256 trueStart = (rangeStartPrice * ONE) / dailyIR;

219: uint256 trueStart = (newPrevRangeClosePrice * ONE) / newDailyIR;

266: uint256 elapsedDays = (currentTime - currentRange.start) / DAY;
```

    diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
    index 03aa7d6..88665b9 100644
    --- a/contracts/rwaOracles/RWADynamicOracle.sol
    +++ b/contracts/rwaOracles/RWADynamicOracle.sol
    @@ -114,8 +114,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
           rangeList[i] = ranges[i];
         }
         if (startTime == ranges[0].start) {
    -      uint256 trueStart = (rangeStartPrice * ONE) / dailyIR;
    -      rangeList[length] = Range(startTime, endTime, dailyIR, trueStart);
    +      rangeList[length] = Range(startTime, endTime, dailyIR, (rangeStartPrice * ONE) / dailyIR);
         } else {
           Range memory lastRange = ranges[ranges.length - 1];
           uint256 prevClosePrice = derivePrice(lastRange, lastRange.end - 1);
    @@ -216,8 +215,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {

         // Update range
         if (indexToModify == 0) {
    -      uint256 trueStart = (newPrevRangeClosePrice * ONE) / newDailyIR;
    -      ranges[indexToModify] = Range(newStart, newEnd, newDailyIR, trueStart);
    +      ranges[indexToModify] = Range(newStart, newEnd, newDailyIR, (newPrevRangeClosePrice * ONE) / newDailyIR);
         } else {
           ranges[indexToModify] = Range(
             newStart,
    @@ -263,11 +261,10 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
         Range memory currentRange,
         uint256 currentTime
       ) internal pure returns (uint256 price) {
    -    uint256 elapsedDays = (currentTime - currentRange.start) / DAY;
         return
           roundUpTo8(
             _rmul(
    -          _rpow(currentRange.dailyInterestRate, elapsedDays + 1, ONE),
    +          _rpow(currentRange.dailyInterestRate, (currentTime - currentRange.start) / DAY + 1, ONE),
               currentRange.prevRangeClosePrice
             )
           );

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-04] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 8 instances of this issue in 4 files:

```
File: contracts/bridge/SourceBridge.sol

164: for (uint256 i = 0; i < exCallData.length; ++i) {
```

    diff --git a/contracts/bridge/SourceBridge.sol b/contracts/bridge/SourceBridge.sol
    index 457bf41..1322d59 100644
    --- a/contracts/bridge/SourceBridge.sol
    +++ b/contracts/bridge/SourceBridge.sol
    @@ -161,7 +161,7 @@ contract SourceBridge is Ownable, Pausable, IMulticall {
         ExCallData[] calldata exCallData
       ) external payable override onlyOwner returns (bytes[] memory results) {
         results = new bytes[](exCallData.length);
    -    for (uint256 i = 0; i < exCallData.length; ++i) {
    +    for (uint256 i = exCallData.length - 1; i >=0 ; --i) {
           (bool success, bytes memory ret) = address(exCallData[i].target).call{
             value: exCallData[i].value
           }(exCallData[i].data);

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol

```
File: contracts/bridge/DestinationBridge.sol

134: for (uint256 i = 0; i < thresholds.length; ++i) {

160: for (uint256 i = 0; i < t.approvers.length; ++i) {

264: for (uint256 i = 0; i < amounts.length; ++i) {
```

    diff --git a/contracts/bridge/DestinationBridge.sol b/contracts/bridge/DestinationBridge.sol
    index 8ad410c..8240e21 100644
    --- a/contracts/bridge/DestinationBridge.sol
    +++ b/contracts/bridge/DestinationBridge.sol
    @@ -131,7 +131,7 @@ contract DestinationBridge is
         string memory srcChain
       ) internal {
         Threshold[] memory thresholds = chainToThresholds[srcChain];
    -    for (uint256 i = 0; i < thresholds.length; ++i) {
    +    for (uint256 i = thresholds.length - 1; i >= 0; --i) {
           Threshold memory t = thresholds[i];
           if (amount <= t.amount) {
             txnToThresholdSet[txnHash] = TxnThreshold(
    @@ -157,7 +157,7 @@ contract DestinationBridge is
         // Check that the approver has not already approved
         TxnThreshold storage t = txnToThresholdSet[txnHash];
         if (t.approvers.length > 0) {
    -      for (uint256 i = 0; i < t.approvers.length; ++i) {
    +      for (uint256 i = t.approvers.length - 1; i >= 0; --i) {
             if (t.approvers[i] == msg.sender) {
               revert AlreadyApproved();
             }
    @@ -261,7 +261,7 @@ contract DestinationBridge is
           revert ArrayLengthMismatch();
         }
         delete chainToThresholds[srcChain];
    -    for (uint256 i = 0; i < amounts.length; ++i) {
    +    for (uint256 i = amounts.length - 1; i >= 0; --i) {
           if (i == 0) {
             chainToThresholds[srcChain].push(
           Threshold(amounts[i], numOfApprovers[i])

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

```
File: contracts/usdy/rUSDYFactory.sol

130: for (uint256 i = 0; i < exCallData.length; ++i) {
```

    diff --git a/contracts/usdy/rUSDYFactory.sol b/contracts/usdy/rUSDYFactory.sol
    index 62178a5..068b16b 100644
    --- a/contracts/usdy/rUSDYFactory.sol
    +++ b/contracts/usdy/rUSDYFactory.sol
    @@ -127,7 +127,7 @@ contract rUSDYFactory is IMulticall {
         ExCallData[] calldata exCallData
       ) external payable override onlyGuardian returns (bytes[] memory results) {
         results = new bytes[](exCallData.length);
    -    for (uint256 i = 0; i < exCallData.length; ++i) {
    +    for (uint256 i = exCallData.length - 1; i >= 0; --i) {
           (bool success, bytes memory ret) = address(exCallData[i].target).call{
             value: exCallData[i].value
           }(exCallData[i].data);

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol


```
File: contracts/rwaOracles/RWADynamicOracle.sol

77: for (uint256 i = 0; i < length; ++i) {

113: for (uint256 i = 0; i < length; ++i) {

129: for (uint256 i = 0; i < length + 1; ++i) {
```

    diff --git a/contracts/rwaOracles/RWADynamicOracle.sol b/contracts/rwaOracles/RWADynamicOracle.sol
    index 03aa7d6..8d5b3b1 100644
    --- a/contracts/rwaOracles/RWADynamicOracle.sol
    +++ b/contracts/rwaOracles/RWADynamicOracle.sol
    @@ -74,7 +74,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
        */
       function getPrice() public view whenNotPaused returns (uint256 price) {
         uint256 length = ranges.length;
    -    for (uint256 i = 0; i < length; ++i) {
    +    for (uint256 i = length - 1; i >= 0 ; --i) {
           Range storage range = ranges[(length - 1) - i];
           if (range.start <= block.timestamp) {
             if (range.end <= block.timestamp) {
    @@ -110,7 +110,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
       ) external view returns (uint256 price) {
         uint256 length = ranges.length;
         Range[] memory rangeList = new Range[](length + 1);
    -    for (uint256 i = 0; i < length; ++i) {
    +    for (uint256 i = length - 1; i >= 0; --i) {
           rangeList[i] = ranges[i];
         }
         if (startTime == ranges[0].start) {
    @@ -126,7 +126,7 @@ contract RWADynamicOracle is IRWAOracle, AccessControlEnumerable, Pausable {
             prevClosePrice
           );
         }
    -    for (uint256 i = 0; i < length + 1; ++i) {
    +    for (uint256 i = length; i >= 0  ; --i) {
           Range memory range = rangeList[(length) - i];
           if (range.start <= blockTimeStamp) {
             if (range.end <= blockTimeStamp) {
            
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }

    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }

    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-05] Revert Transaction as soon as possible

Always try reverting transactions as early as possible when using require statements . In case a transaction revert occurs, the user will pay the gas up until the revert was executed

There is 1 instance of this issue in  1 file:

```
File: contracts/bridge/SourceBridge.sol

61: function burnAndCallAxelar(
62:   uint256 amount,
63:   string calldata destinationChain
64: ) external payable whenNotPaused {
65:   // check destinationChain is correct
66:   string memory destContract = destChainToContractAddr[destinationChain];
67:
68:  if (bytes(destContract).length == 0) {
69:     revert DestinationNotSupported();
70:   }
71:
72:  if (msg.value == 0) {
73:     revert GasFeeTooLow();
74:   }
```
    diff --git a/contracts/bridge/SourceBridge.sol b/contracts/bridge/SourceBridge.sol
    index 457bf41..678a461 100644
    --- a/contracts/bridge/SourceBridge.sol
    +++ b/contracts/bridge/SourceBridge.sol
    @@ -62,6 +62,10 @@ contract SourceBridge is Ownable, Pausable, IMulticall {
         uint256 amount,
         string calldata destinationChain
       ) external payable whenNotPaused {
    +    if (msg.value == 0) {
    +      revert GasFeeTooLow();
    +    }
    +
         // check destinationChain is correct
         string memory destContract = destChainToContractAddr[destinationChain];

    @@ -69,10 +73,6 @@ contract SourceBridge is Ownable, Pausable, IMulticall {
           revert DestinationNotSupported();
         }

    -    if (msg.value == 0) {
    -      revert GasFeeTooLow();
    -    }
    -
         // burn amount
         TOKEN.burnFrom(msg.sender, amount);

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol