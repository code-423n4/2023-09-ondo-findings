## Impact
Missing constructor sanity checks

The implementation of constructor() functions are missing some sanity checks.


## Proof of Concept

Missing sanity check for != address(0)

There're no sanity checks for the constructor argument _guardian.

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol?plain=1#L51-L53
```
  constructor(address _guardian) {
    guardian = _guardian;
  }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol?plain=1#L30-L46
```
  constructor(
    address admin,
    address setter,
    address pauser,
    uint256 firstRangeStart,
    uint256 firstRangeEnd,
    uint256 dailyIR,
    uint256 startPrice
  ) {
    _grantRole(DEFAULT_ADMIN_ROLE, admin);
    _grantRole(PAUSER_ROLE, pauser);
    _grantRole(SETTER_ROLE, setter);

    if (firstRangeStart >= firstRangeEnd) revert InvalidRange();
    uint256 trueStart = (startPrice * ONE) / dailyIR;
    ranges.push(Range(firstRangeStart, firstRangeEnd, dailyIR, trueStart));
  }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol?plain=1#L40-L50
```
  constructor(
    address _token,
    address _axelarGateway,
    address _gasService,
    address owner
  ) {
    TOKEN = IRWALike(_token);
    AXELAR_GATEWAY = IAxelarGateway(_axelarGateway);
    GAS_RECEIVER = IAxelarGasService(_gasService);
    _transferOwnership(owner);
  }
```

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol?plain=1#L55-L72
```
  constructor(
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
    TOKEN = IRWALike(_token);
    AXELAR_GATEWAY = IAxelarGateway(_axelarGateway);
    ALLOWLIST = IAllowlist(_allowlist);
    approvers[_ondoApprover] = true;
    _transferOwnership(_owner);
  }
```


## Tools Used
Manual code review

## Recommended Mitigation Steps
To reduce possible errors and make the code more robust, consider adding sanity checks where needed.