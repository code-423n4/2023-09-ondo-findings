
# Gas Optimizations

| Number | Issue | Instances | Gas |
|--------|-------|-----------|-----|
|[G-01]| Don’t cache value if it is only used once | 5  |  450 |
|[G-02]| Use assembly to check for address(0) | 12  | 72 |
|[G-03]| Use hardcode address instead address(this) | 3  |   |
|[G-04]| Make 3 event parameters indexed when possible | 7  | 3500 |
|[G-05]| Use assembly to write address storage values | 3 | 600 |
|[G-06]| Assigning keccak operations to constant variables results in extra gas costs | 7 |   |
|[G-07]| A mapping is more efficient than an array | 1 | 248 |
|[G-08]| Public Functions not called by they contract should be declear External | 10  |   |
|[G-09]| Save loop calls | 3 |   |
|[G-10]| Create immutable variable to avoid redundant external calls | 2 |   |
|[G-11]| Use assembly to validate msg.sender | 6  | 18  |
|[G-12]| Cache calldata/memory pointers for complex types to avoid offset calculations | 2  |   |
|[G-13]| Combine events to save 2 Glogtopic (375 gas) | 3  | 1125 |
|[G-14]| Internal/Private functions only called once can be inlined to save gas Gas saved: 40 | 1 | 40 |
|[G-15]| bytes constants are more eficient than string constans | 19 |   |
|[G-16]| Amounts should be checked for 0 before calling a transfer | 6 |   |
|[G-17]| abi.encode() is less efficient than abi.encodePacked() | 3  |   |
|[G-18]| Do not calculate constants | 1 |   |
|[G-19]| Use assembly for loops | 6 |   |
|[G-20]| Avoid emitting storage values | 1 |   |
|[G-21]| Use assembly to make more efficient back-to-back calls | 1 |   |
|[G-22]| Use a do while loop instead of a for loop | 4 |   |
|[G-23]| State variables can be packed into fewer storage slots | 1  | 2100 |
|[G-24]| Cache state variables outside of loop to avoid reading storage on every iteration | 1 |   |
|[G-25]| Use calldata instead of memory for function parameters that don’t change | 3  | 900 |
|[G-26]| Don’t initialize variables with default value | 8  | 46  |
|[G-27]| Use uint256(1)/uint256(2) instead for true and false boolean states | 3 | 51300 |

## [G-01] Don’t cache value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation

### Don't cache they value of  IRWALike(_token).balanceOf(address(this)); because the used only one

```solidity
file:  contracts/bridge/DestinationBridge.sol

322     function rescueTokens(address _token) external onlyOwner {
    uint256 balance = IRWALike(_token).balanceOf(address(this));
    IRWALike(_token).transfer(owner(), balance);
  }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L322-L325>

### Don't cache they value of _checkThresholdMet(txnHash); because the used only one

```solidity
file:  contracts/bridge/DestinationBridge.sol
 
337    function _mintIfThresholdMet(bytes32 txnHash) internal {
    bool thresholdMet = _checkThresholdMet(txnHash);
    Transaction memory txn = txnHashToTransaction[txnHash];
    if (thresholdMet) {
      _checkAndUpdateInstantMintLimit(txn.amount);
      if (!ALLOWLIST.isAllowed(txn.sender)) {
        ALLOWLIST.setAccountStatus(
          txn.sender,
          ALLOWLIST.getValidTermIndexes()[0],
          true
        );
      }
      TOKEN.mint(txn.sender, txn.amount);
      delete txnHashToTransaction[txnHash];
      emit BridgeCompleted(txn.sender, txn.amount);
    }
  }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L337-L353>

### Don't cache they value of abi.encode(VERSION, msg.sender, amount, nonce++); because the used only one

```solidity
file:  contracts/bridge/SourceBridge.sol

79   bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

    _payGasAndCallContract(destinationChain, destContract, payload);
  }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79-L81>

### Don't cache they value of (currentTime - currentRange.start) / DAY; because the used only one

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.sol
 
262    function derivePrice(
    Range memory currentRange,
    uint256 currentTime
  ) internal pure returns (uint256 price) {
    uint256 elapsedDays = (currentTime - currentRange.start) / DAY;
    return
      roundUpTo8(
        _rmul(
          _rpow(currentRange.dailyInterestRate, elapsedDays + 1, ONE),
          currentRange.prevRangeClosePrice
        )
      );
  }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L262-L274>

### Don't cache they value of getRUSDYByShares(_sharesAmount); and because the used only one

```solidity
file:    contracts/usdy/rUSDY.sol

575    function _burnShares(
    address _account,
    uint256 _sharesAmount
  ) internal whenNotPaused returns (uint256) {
    require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

    _beforeTokenTransfer(_account, address(0), _sharesAmount);

    uint256 accountShares = shares[_account];
    require(_sharesAmount <= accountShares, "BURN_AMOUNT_EXCEEDS_BALANCE");

    uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

    totalShares -= _sharesAmount;

    shares[_account] = accountShares - _sharesAmount;

    uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

    emit SharesBurnt(
      _account,
      preRebaseTokenAmount,
      postRebaseTokenAmount,
      _sharesAmount
    );

    return totalShares;

    // Notice: we're not emitting a Transfer event to the zero address here since shares burn
    // works by redistributing the amount of tokens corresponding to the burned shares between
    // all other token holders. The total supply of the token doesn't change as the result.
    // This is equivalent to performing a send from `address` to each other token holder address,
    // but we cannot reflect this as it would require sending an unbounded number of events.

    // We're emitting `SharesBurnt` event to provide an explicit rebase log record nonetheless.
  }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L575-L610>

## [G-02] Use assembly to check for address(0)

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression; especially if the check is performed frequently or in a loop. However, it’s important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.

```solidity
file:   contracts/usdy/rUSDY.sol
 
238      emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));

439      emit TransferShares(address(0), msg.sender, _USDYAmount);

490      require(_owner != address(0), "APPROVE_FROM_ZERO_ADDRESS");

490      require(_spender != address(0), "APPROVE_TO_ZERO_ADDRESS");

519      require(_sender != address(0), "TRANSFER_FROM_THE_ZERO_ADDRESS");

520      require(_recipient != address(0), "TRANSFER_TO_THE_ZERO_ADDRESS");

547      require(_recipient != address(0), "MINT_TO_THE_ZERO_ADDRESS");

549     _beforeTokenTransfer(address(0), _recipient, _sharesAmount);

579      require(_account != address(0), "BURN_FROM_THE_ZERO_ADDRESS");

580     _beforeTokenTransfer(_account, address(0), _sharesAmount);

642      if (from != address(0)) {

649      if (to != address(0)) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L238>

## [G-03] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```solidity
file:  contracts/bridge/DestinationBridge.sol

323   uint256 balance = IRWALike(_token).balanceOf(address(this));

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L323>

```solidity
file:  contracts/bridge/SourceBridge.sol

97     address(this),

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L97>

```solidity
file:  contracts/usdy/rUSDY.sol

437    usdy.transferFrom(msg.sender, address(this), _USDYAmount);

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L437>

## [G-04] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file:  contracts/bridge/DestinationBridge.sol

389    event ApproverRemoved(address approver);

396    event ApproverAdded(address approver);

405    event ChainIdSupported(string srcChain, string approvedSource);

414    event ThresholdSet(
        string chain,
        uint256[] amounts,
        uint256[] numOfApprovers
    );
  
426   event BridgeCompleted(address user, uint256 amount);

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L389>

```solidity
file:  contracts/usdy/rUSDY.sol

189    event TokensBurnt(address indexed account, uint256 tokensBurnt);

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L189>

```solidity
file:  contracts/bridge/SourceBridge.sol

183    event DestinationChainContractAddressSet(
    string indexed destinationChain,
    address contractAddress
  );

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L183>

## [G-05] Use assembly to write address storage values

### Mitigation

Use assembly to write address storage values. Here are a few reasons:

Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs.
Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings.
Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space.
Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.

```solidity
file:  contracts/bridge/DestinationBridge.sol

55     constructor(
    address _token,
    address _axelarGateway,
    address _allowlist,
    address _ondoApprover,
    address _owner,
    uint256 _mintLimit,
    uint256 _mintDuration
  )
    AxelarExecutable(_axelarGateway)
    MintTimeBasedRateLimiter(_mintDuration, _mintLimit)
  {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L55-L66>

```solidity
file:  contracts/bridge/SourceBridge.sol

40     constructor(
    address _token,
    address _axelarGateway,
    address _gasService,
    address owner
  ) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L40-L45>

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.sol

30      constructor(
    address admin,
    address setter,
    address pauser,
    uint256 firstRangeStart,
    uint256 firstRangeEnd,
    uint256 dailyIR,
    uint256 startPrice
  ) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L30-L38>

## [G-06] Assigning keccak operations to constant variables results in extra gas costs

"constants" expressions are expressions. As such, keccak assigned to a constant variable are re-computed at each use of the variable, which will consume gas unnecessarily. To save gas, Changing the variable from constant to immutable will make the computation run only once and therefore save gas.

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.sol

27     bytes32 public constant SETTER_ROLE = keccak256("SETTER_ROLE");

28     bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L27>

```solidity
file:  contracts/usdy/rUSDY.sol

97     bytes32 public constant USDY_MANAGER_ROLE = keccak256("ADMIN_ROLE");

98     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

99     bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

100    bytes32 public constant BURNER_ROLE = keccak256("BURN_ROLE");

101    bytes32 public constant LIST_CONFIGURER_ROLE = keccak256("LIST_CONFIGURER_ROLE");

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L97>

### Recommended Mitigation Steps

Change the variable from constant to immutable

## [G-07] A mapping is more efficient than an array

Fetching data from an array is more expensive than fetching data from a mapping. Fetching data from an array will require iterating over the array until ou reach your desired data. When using a mapping you only need to know the key in order to fetch the data from the exact slot it is stored in

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.

25   Range[] public ranges

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L25>

## [G-08] Public Functions not called by they contract should be declear External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity
file:  contracts/usdy/rUSDY.sol

109     function initialize(
    address blocklist,
    address allowlist,
    address sanctionsList,
    address _usdy,
    address guardian,
    address _oracle
  ) public virtual initializer {

194   function name() public pure returns (string memory) {

202   function symbol() public pure returns (string memory) {

209   function decimals() public pure returns (uint8) {

216   function totalSupply() public view returns (uint256) {

226   function balanceOf(address _account) public view returns (uint256) {

276   function approve(address _spender, uint256 _amount) public returns (bool) {

372   function getTotalShares() public view returns (uint256) {

381   function sharesOf(address _account) public view returns (uint256) {

416     function transferShares(
    address _recipient,
    uint256 _sharesAmount
  ) public returns (uint256) {
  

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L109-L116>

## [G-09] Save loop calls

### Instead of calling amounts[i] 3 times in each loop for fetching data, it can be saved as a variable

```solidity
file:  contracts/bridge/DestinationBridge.sol

264      for (uint256 i = 0; i < amounts.length; ++i) {
      if (i == 0) {
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      } else {
        if (chainToThresholds[srcChain][i - 1].amount > amounts[i]) {
          revert ThresholdsNotInAscendingOrder();
        }
        chainToThresholds[srcChain].push(
          Threshold(amounts[i], numOfApprovers[i])
        );
      }
    }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L264-L277>

### Instead of calling exCallData[i] 3 times in each loop for fetching data, it can be saved as a variable

```solidity
file:  contracts/bridge/SourceBridge.sol

164     for (uint256 i = 0; i < exCallData.length; ++i) {
      (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
      require(success, "Call Failed");
      results[i] = ret;
    }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L164-L170>

### Instead of calling exCallData[i] 3 times in each loop for fetching data, it can be saved as a variable

```solidity
file:  contracts/usdy/rUSDYFactory.sol

130    for (uint256 i = 0; i < exCallData.length; ++i) {
      (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
      require(success, "Call Failed");
      results[i] = ret;
    }
  
```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L130-L136>

## [G-10] Create immutable variable to avoid redundant external calls

```solidity
file:  contracts/usdy/rUSDY.sol

88   IUSDY public usdy;

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L88>

```solidity
file:  rwaOracles/RWADynamicOracle.sol

25    Range[] public ranges;

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L25>

## [G-11] Use assembly to validate msg.sender

```solidity
file:  contracts/usdy/rUSDYFactory.sol

155   require(msg.sender == guardian, "rUSDYFactory: You are not the Guardian");

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L155>

```solidity
file:  contracts/usdy/rUSDY.sol

633   if (from != msg.sender && to != msg.sender) {

634   require(!_isBlocked(msg.sender), "rUSDY: 'sender' address blocked");

635   require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");

636   require(
        _isAllowed(msg.sender),
        "rUSDY: 'sender' address not on allowlist"
      );
```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L633>

```solidity
file:  contracts/bridge/DestinationBridge.sol

161   if (t.approvers[i] == msg.sender) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L161>

## [G-12] Cache calldata/memory pointers for complex types to avoid offset calculations

The function parameters in the following instances are complex types (i.e. arrays which contain structs) and thus will result in more complex offset calculations to retrieve specific data from calldata/memory. We can avoid peforming some of these offset calculations by instantiating calldata/memory pointers.

```solidity
file:  contracts/bridge/SourceBridge.sol

160     function multiexcall(
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

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L160-L171>

```solidity
file:  contracts/usdy/rUSDYFactory.sol

126     function multiexcall(
    ExCallData[] calldata exCallData
  ) external payable override onlyGuardian returns (bytes[] memory results) {
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

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L126-L137>

## [G-13] Combine events to save 2 Glogtopic (375 gas)

The events below are only emitted once in the handleRewards function. We can combine the events into one singular event to save two Glogtopic (375 gas) that would otherwise be paid for the additional two events.

```solidity
file: contracts/usdy/rUSDY.sol

420    _transferShares(msg.sender, _recipient, _sharesAmount);
       emit TransferShares(msg.sender, _recipient, _sharesAmount);
       uint256 tokensAmount = getRUSDYByShares(_sharesAmount);
       emit Transfer(msg.sender, _recipient, tokensAmount);
       return tokensAmount;
  }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L420-L425>

```solidity
file:   contracts/usdy/rUSDY.sol

436     _mintShares(msg.sender, _USDYAmount * BPS_DENOMINATOR);
        usdy.transferFrom(msg.sender, address(this), _USDYAmount);
        emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
        emit TransferShares(address(0), msg.sender, _USDYAmount);

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L436-L239>

```solidity
file:  contracts/usdy/rUSDY.sol

468     uint256 _sharesToTransfer = getSharesByRUSDY(_amount);
       _transferShares(_sender, _recipient, _sharesToTransfer);
       emit Transfer(_sender, _recipient, _amount);
       emit TransferShares(_sender, _recipient, _sharesToTransfer);
       
```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L468-L471>

## [G-14] Internal/Private functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
file:  contracts/bridge/SourceBridge.sol

91     function _payGasAndCallContract(
    string calldata destinationChain,
    string memory destContract,
    bytes memory payload
  ) private {


```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L91-L95>

## [G-15] bytes constants are more eficient than string constans

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
file: contracts/bridge/DestinationBridge.sol

52   mapping(string => Threshold[]) public chainToThresholds;

86   string calldata srcChain,

87   string calldata srcAddr,

131  string memory srcChain

235  string calldata srcChain,

236  string calldata srcContractAddres

256  string calldata srcChain,

405  event ChainIdSupported(string srcChain, string approvedSource);

414  event ThresholdSet(string chain, uint256[] amounts, uint256[] numOfApprovers);

433   string srcChain,

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L52>

```solidity
file:  contracts/bridge/SourceBridge.sol

15   mapping(string => string) public destChainToContractAddr;

63   string calldata destinationChain

66   string memory destContract = destChainToContractAddr[destinationChain];

92   string calldata destinationChain,

93   string memory destContract

122  string memory destinationChain,

184  string indexed destinationChain,

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L15>

```solidity
file:   contracts/usdy/rUSDYFactory.sol

150     string name,

151     string ticker

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L150>

## [G-16] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

```solidity
file: contracts/usdy/rUSDY.sol

246    _transfer(msg.sender, _recipient, _amount);

309     _transfer(msg.sender, _recipient, _amount);

420    _transferShares(msg.sender, _recipient, _sharesAmount);

522    _beforeTokenTransfer(_sender, _recipient, _sharesAmount);

549   _beforeTokenTransfer(address(0), _recipient, _sharesAmount);

581   _beforeTokenTransfer(_account, address(0), _sharesAmount);


```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L246>

## [G-17] abi.encode() is less efficient than abi.encodePacked()

Consider changing it if possible.

```solidity
file:  contracts/bridge/DestinationBridge.sol

99    if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {

238   chainToApprovedSender[srcChain] = keccak256(abi.encode(srcContractAddress));

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L99>

```solidity
file:  contracts/bridge/SourceBridge.sol

79     bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L79>

## [G-18] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
file:   contracts/rwaOracles/RWADynamicOracle.sol

343    uint256 private constant ONE = 10 ** 27;

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L343>

## [G-19] Use assembly for loops

We can use assembly to write a more gas efficient loop. See the final diffs for comments regarding the assembly code.

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.sol

113  for (uint256 i = 0; i < length; ++i) {
      rangeList[i] = ranges[i];
    }

129    for (uint256 i = 0; i < length + 1; ++i) {
      Range memory range = rangeList[(length) - i];
      if (range.start <= blockTimeStamp) {
        if (range.end <= blockTimeStamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, blockTimeStamp);
        }
      }
    }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L113-L115>

```solidity
file:   contracts/bridge/DestinationBridge.sol

134       for (uint256 i = 0; i < thresholds.length; ++i) {
      Threshold memory t = thresholds[i];
      if (amount <= t.amount) {
        txnToThresholdSet[txnHash] = TxnThreshold(
          t.numberOfApprovalsNeeded,
          new address[](0)
        );
        break;
      }
    }

160   for (uint256 i = 0; i < t.approvers.length; ++i) {
        if (t.approvers[i] == msg.sender) {
          revert AlreadyApproved();
        }
      }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L134-L143>

```solidity
file:   contracts/bridge/SourceBridge.sol

164     for (uint256 i = 0; i < exCallData.length; ++i) {
      (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
      require(success, "Call Failed");
      results[i] = ret;
    }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L164-L170>

```solidity
file:   contracts/usdy/rUSDYFactory.sol

130       for (uint256 i = 0; i < exCallData.length; ++i) {
      (bool success, bytes memory ret) = address(exCallData[i].target).call{
        value: exCallData[i].value
      }(exCallData[i].data);
      require(success, "Call Failed");
      results[i] = ret;
    }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L130-L136>

## [G-20] Avoid emitting storage values

In the instance below, we can emit the calldata value instead of emitting a storage value. This will result in using a cheap CALLDATALOAD instead of an expensive SLOAD.

```solidity
file:   contracts/usdy/rUSDYFactory.sol

101       emit rUSDYDeployed(
      address(rUSDYProxy),
      address(rUSDYProxyAdmin),
      address(rUSDYImplementation),
      "Ondo Rebasing U.S. Dollar Yield",
      "rUSDY"
    );
```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L101-L107>

## [G-21] Use assembly to make more efficient back-to-back calls

In the instance below, two external calls, both of which take two function parameters, are performed. We can potentially avoid memory expansion costs by using assembly to utilize scratch space + free memory pointer memory offsets for the function calls. We will use the same memory space for both function calls.

```solidity
file:   contracts/usdy/rUSDY.sol    

586      uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

    totalShares -= _sharesAmount;

    shares[_account] = accountShares - _sharesAmount;

    uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L586-L592>

## [G-22] Use a do while loop instead of a for loop

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.sol

113  for (uint256 i = 0; i < length; ++i) {
      rangeList[i] = ranges[i];
    }

129    for (uint256 i = 0; i < length + 1; ++i) {
      Range memory range = rangeList[(length) - i];
      if (range.start <= blockTimeStamp) {
        if (range.end <= blockTimeStamp) {
          return derivePrice(range, range.end - 1);
        } else {
          return derivePrice(range, blockTimeStamp);
        }
      }
    }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L113-L115>

```solidity
file:   contracts/bridge/DestinationBridge.sol

134       for (uint256 i = 0; i < thresholds.length; ++i) {
      Threshold memory t = thresholds[i];
      if (amount <= t.amount) {
        txnToThresholdSet[txnHash] = TxnThreshold(
          t.numberOfApprovalsNeeded,
          new address[](0)
        );
        break;
      }
    }

160   for (uint256 i = 0; i < t.approvers.length; ++i) {
        if (t.approvers[i] == msg.sender) {
          revert AlreadyApproved();
        }
      }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L134-L143>

## [G-23] State variables can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```solidity
file:   contracts/usdy/rUSDY.sol

82     uint256 private totalShares;

  // Address of the oracle that updates `usdyPrice`
  IRWADynamicOracle public oracle;

  // Address of the USDY token
  IUSDY public usdy;

  // Used to scale up usdy amount -> shares
  uint256 public constant BPS_DENOMINATOR = 10_000;


```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol#L82-L91>

## [G-24] Cache state variables outside of loop to avoid reading storage on every iteration

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.

### Cache  txnToThresholdSet[txnHash] outside of the loop to save 1 SLOAD per iteration

```solidity
file:   contracts/bridge/DestinationBridge.sol

134      for (uint256 i = 0; i < thresholds.length; ++i) {
      Threshold memory t = thresholds[i];
      if (amount <= t.amount) {
        txnToThresholdSet[txnHash] = TxnThreshold(
          t.numberOfApprovalsNeeded,
          new address[](0)
        );
        break;
      }
    }

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L134-L143>

## [G-25] Use calldata instead of memory for function parameters that don’t change

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
file:  contracts/bridge/DestinationBridge.sol

128     function _attachThreshold(
    uint256 amount,
    bytes32 txnHash,
    string memory srcChain
  ) internal {


```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L128-L132>

```solidity
file:  contracts/bridge/SourceBridge.sol

91     function _payGasAndCallContract(
    string calldata destinationChain,
    string memory destContract,
    bytes memory payload
  ) private {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L91-L95>

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.sol

262    function derivePrice(
    Range memory currentRange,
    uint256 currentTime
  ) internal pure returns (uint256 price) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L262-L265>

## [G-26] Don’t initialize variables with default value

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.

```solidity
file:  contracts/bridge/DestinationBridge.sol

134   for (uint256 i = 0; i < thresholds.length; ++i) {

160   for (uint256 i = 0; i < t.approvers.length; ++i) {

264   for (uint256 i = 0; i < amounts.length; ++i) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L134>

```solidity
file:  contracts/bridge/SourceBridge.sol

164    for (uint256 i = 0; i < exCallData.length; ++i) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol#L164>

```solidity
file:  contracts/rwaOracles/RWADynamicOracle.sol

77    for (uint256 i = 0; i < length; ++i) {

113   for (uint256 i = 0; i < length; ++i) {

129   for (uint256 i = 0; i < length + 1; ++i) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol#L77>

```solidity
file:   contracts/usdy/rUSDYFactory.sol

130    for (uint256 i = 0; i < exCallData.length; ++i) {

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol#L130>

## [G-36] Use uint256(1)/uint256(2) instead for true and false boolean states

```solidity
file: contracts/bridge/DestinationBridge.sol

70    approvers[_ondoApprover] = true;

106   isSpentNonce[chainToApprovedSender[srcChain]][nonce] = true;

211   approvers[approver] = true;

```

<https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol#L70>
