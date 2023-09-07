# Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | There should be a cap on `dailyInterestRate` | 2 |
| [L-2](#L-2) | `getPrice` can return 0 if startTime is set more than block.timestamp | 1 |
### [L-1] There should be a cap on `dailyInterestRate` 
`dailyInterestRate` is a value set in range by the admin. Since it is being exponentiated by number of days, if 
for any reason that value is not scaled appropriately, or interest is passed for a monthly or yearly rates instead
of daily, the resulting price would be way off the expected value.
<br><br>
Consider adding a `MIN_INTEREST` and `MAX_INTEREST` values to make sure the interest rate is not accidently passed
incorrectly. 
``` solidity
require( dailyInterestRate <= MAX_INTEREST );
require( dailyInterestRate >= MIN_INTEREST );
```

Contracts in scope: https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

*Instances (2)*: <br>
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L151
```solidity
  function setRange(
    uint256 endTimestamp,
    uint256 dailyInterestRate
  ) external onlyRole(SETTER_ROLE) {}
```
https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L186
``` solidity
function overrideRange(
    uint256 indexToModify,
    uint256 newStart,
    uint256 newEnd,
    uint256 newDailyIR,
    uint256 newPrevRangeClosePrice
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {}
```
### [L-2] `getPrice` can return 0 if startTime is set more than block.timestamp
The first rage is set in the constructor, but there is no limit as to how long in the future the range can start. This
means that if `firstRangeStart` is more than block.timestamp (during deployment) then anyone can call the public function
`getPrice` which would return 0 unless `firstRangeStart > block.timestamp` .
<br><br>
Since this function is crucial in determining the amount of rUSDY tokens of the user, there must be checks in place to make
sure that price returned is never 0. The contract can also be immediatly after deployement and shouldn't be unpaused until
`firstrangeStart` is hit. 

Instances (1)

``` solidity
function getPrice() public view whenNotPaused returns (uint256 price) {
    uint256 length = ranges.length;
    for (uint256 i = 0; i < length; ++i) {
      Range storage range = ranges[(length - 1) - i];
      if (range.start <= block.timestamp) {
        if (range.end <= block.timestamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, block.timestamp);
        }
      }
    }
  }
```


# Non-Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [N-1](#N-1) | Low level call can return true if address is EOA | 2 |
| [N-2](#N-2) | No checks for address(0) | 2 |
### [N-1] Low level call can return true if address is EOA
A low level call in solidity to an address that is not a contract would still return true. So if you're passing calldata
and for some reason the contract address is incorrect, it'll still return true but nothing will happen. 
<br><br>
`multiexcall` function in both `rUSDYFactory.sol` and `SourceBridge.sol` make low level calls to an array of addresses but
dont make sure if they're correct or not. 
<br><br>
When call data is not 0, make sure address you're calling is a contract address.

``` solidity
modifier isNotContract(address addr) {
  uint size;
  assembly {
    size := extcodesize(addr)
  }
    require(size == 0);
     _;
}
```

Contracts in scope: https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L126

Contracts in scope: https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L160

### [N-2] No checks for address(0)
These are instances missed by bot report where address called is not checked for address(0).

``` solidity
require( exCallData[i].target != address(0) )
```

Instances (2)

Contracts in scope: https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L126

Contracts in scope: https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L160

``` solidity
  function multiexcall(
    ExCallData[] calldata exCallData
  ) external payable override onlyOwner returns (bytes[] memory results) {
    results = new bytes[](exCallData.length);
    for (uint256 i = 0; i < exCallData.length; ++i) {
      (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
      require(success, "Call Failed");
      results[i] = ret;
    }
  }

```