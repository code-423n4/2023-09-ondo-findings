## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 2 | - |
| [G-02] | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 1 | - |
| [G-03] | Using fixed bytes is cheaper than using string | 2 | - |
| [G-04] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 7 | - |
| [G-05] | Not using the named return variable when a function returns, wastes deployment gas | 2 | - |
| [G-06] | Should use arguments instead of state variable | 4 | - |
| [G-07] | Before transfer of  some functions, we should check some variables for possible gas save | 2 | - |
| [G-08] | With assembly, .call (bool success)  transfer can be done gas-optimized | 2 | - |
| [G-09] | Duplicated require()/if() checks should be refactored to a modifier or function | 1 | - |
| [G-10] | A modifier used only once and not being inherited should be inlined to save gas | 1 | - |
| [G-11] | abi.encode() is less efficient than  abi.encodepacked() | 3 | - |
| [G-12] | Do not calculate  constant | 1 | - |
| [G-13] | require() Should Be Used Instead Of assert() | 1 | - |
| [G-14] | Use hardcode address instead address(this) | 3 | - |
| [G-15] | Use assembly for math (add, sub, mul, div) | 3 | - |
| [G-16] | Multiplication/division by two should use bit shifting | 1 | - |
| [G-17] | Refactor event to avoid emitting empty data | 1 | - |
| [G-18] | 2**<N> should be re-written as type(uint<N>).max | 1 | - |
| [G-19] | Shorten the array rather than copying to a new one | 1 | - |
| [G-20] | Use assembly to validate msg.sender | 6 | - |
| [G-21] | No need to evaluate all expressions to know if one of them is true | 1 | - |
| [G-22] | Make 3 event parameters indexed when possible | 3 | - |

## Gas Optimizations  

## [G-01] Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
file: /contracts/bridge/SourceBridge.sol

125    destChainToContractAddr[destinationChain] = AddressToString.toString(
126      contractAddress
127    );

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L125-L127


```solidity
file: /contracts/bridge/DestinationBridge.sol

323    uint256 balance = IRWALike(_token).balanceOf(address(this));

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323


## [G-02] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
       
  Saves 5 gas per iteration

```solidity
file: /contracts/bridge/SourceBridge.sol

79    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);     //FOUND  nonce++

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79


## [G-03] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
file: /contracts/usdy/rUSDY.sol

///@audit "rUSDY"  is 5 bytes : Recommended -- > (bytes5 memory)

202  function symbol() public pure returns (string memory) {
203    return "rUSDY";

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L202-L203

```solidity
file: /contracts/bridge/DestinationBridge.sol

236    string calldata srcContractAddress

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L236


## [G-04] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor. 

```solidity
file: /contracts/usdy/rUSDY.sol

97     bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");
98     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
99     bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
100    bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");
101    bytes32 public constant LIST_CONFIGURER_ROLE =
102     keccak256("LIST_CONFIGURER_ROLE");

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97-L102

```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

27  bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");
28  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L27-L28


## [G-05] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.

```solidity
file: /contracts/usdy/rUSDYFactory.sol

81  ) external onlyGuardian returns (address, address, address) {

/// @audit the ' address ' data type should not used in the return, because waste extra gas its write in fanction return 
///  on line 81

108    return (
      address(rUSDYProxy),
      address(rUSDYProxyAdmin),
      address(rUSDYImplementation)
112    );

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L108-L112


### Recommended conde

``` solidity

81  ) external onlyGuardian returns (address, address, address) {

change to:

108    return (
      rUSDYProxy,
      rUSDYProxyAdmin,
      rUSDYImplementation
112    );

```

```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

///@audit  the ' price ' and ' timestamp ' should not be mentioned in returns,  these will consume extra gas because these
/// varaibles are used in body.

61    returns (uint256 price, uint256 timestamp)
  {
    price = getPrice();
    timestamp = block.timestamp;
  }
 
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L61-L65

## [G-06] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity
file: /contracts/usdy/rUSDYFactory.sol

///@audit the ' (rUSDYProxy),(rUSDYProxyAdmin),rUSDYImplementation), ' are state variables,should emit stack instead of are

101    emit rUSDYDeployed(
      address(rUSDYProxy),
      address(rUSDYProxyAdmin),
      address(rUSDYImplementation),
      "Ondo Rebasing U.S. Dollar Yield",
      "rUSDY"
107    );

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L101-L107


## [G-07] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything

```solidity
file: /contracts/bridge/DestinationBridge.sol

324    IRWALike(_token).transfer(owner(), balance);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L324

```solidity
file: /contracts/usdy/rUSDY.sol

680    usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L680


## [G-08] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), 
this storage disappears and gas optimization is provided.

```solidity
file: /contracts/bridge/SourceBridge.sol

165      (bool success, bytes memory ret) = address(exCallData[i].target).call{

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L165


```solidity
file: /contracts/usdy/rUSDYFactory.sol

131      (bool success, bytes memory ret) = address(exCallData[i].target).call{

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L131


## [G-09] Duplicated require()/if() checks should be refactored to a modifier or function   

to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

/// @audit the duplicated if condition is on line 218

198    if (indexToModify == 0) {

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L198


## [G-10] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: /contracts/usdy/rUSDYFactory.sol

154  modifier onlyGuardian() {
     require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");
     _;
157  }

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L154-L157


## [G-11] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/bridge/DestinationBridge.sol

99    if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {

238    chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));    

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L99


```solidity
file: /contracts/bridge/SourceBridge.sol

79    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79


## [G-12] Do not calculate  constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

343  uint256 private constant ONE = 10 ** 27;

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L343


## [G-13] require() Should Be Used Instead Of assert()

```solidity
file: /contracts/usdy/rUSDYFactory.sol

100    assert(rUSDYProxyAdmin.owner() == guardian);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L100


## [G-14] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

```solidity
file: /contracts/bridge/SourceBridge.sol

97      address(this),

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L97


```solidity
file: /contracts/bridge/DestinationBridge.sol

323    uint256 balance = IRWALike(_token).balanceOf(address(this));

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323


```solidity
file: /contracts/usdy/rUSDY.sol

437    usdy.transferFrom(msg.sender, address(this), _USDYAmount);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L437


## [G-15] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

266    uint256 elapsedDays = (currentTime - currentRange.start) / DAY;

283    uint256 remainder = value % 1e10;

401    z = _mul(x, y) / ONE;

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L266


## [G-16] Multiplication/division by two should use bit shifting

<x> * 2 is the same as <x> << 1. While the compiler uses the SHL opcode to accomplish both, the version that uses multiplication incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.

```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

405    require(y == 0 || (z = x * y) / y == x);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L405


## [G-17] Refactor event to avoid emitting empty data

```solidity
file: /contracts/usdy/rUSDYFactory.sol

87      ""

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L87


## [G-18] 2**<N> should be re-written as type(uint<N>).max

Earlier versions of solidity can use uint<n>(-1) instead. Expressions not including the - 1 can often be re-written to accomodate the change (e.g. by using a > rather than a >=, which will also save some gas)

```solidity 
file: /contracts/rwaOracles/RWADynamicOracle.sol
 
343  uint256 private constant ONE = 10 ** 27;

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L343


## [G-19] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

112    Range[] memory rangeList = new Range[](length + 1);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L112


## [G-20] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: /contracts/bridge/DestinationBridge.sol

161        if (t.approvers[i] == msg.sender) {

166    t.approvers.push(msg.sender);

198    if (!approvers[msg.sender]) {

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L161


```solidity
file: /contracts/usdy/rUSDY.sol

420    _transferShares(msg.sender, _recipient, _sharesAmount);

423    emit Transfer(msg.sender, _recipient, tokensAmount);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L420


```solidity
file: /contracts/usdy/rUSDYFactory.sol

155    require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L155


## [G-21] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved
 
```solidity
file: /contracts/rwaOracles/RWADynamicOracle.sol

405    require(y == 0 || (z = x * y) / y == x);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L405


## [G-22] Make 3 event parameters indexed when possible

It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file: /contracts/bridge/SourceBridge.sol

183   event DestinationChainContractAddressSet(
       string indexed destinationChain,
       address contractAddress
186   );

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L183-L186


```solidity
file: /contracts/usdy/rUSDY.sol

154   event TransferShares(
       address indexed from,
       address indexed to,
       uint256 sharesValue
158   );


189  event TokensBurnt(address indexed account, uint256 tokensBurnt);

```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L154-L158