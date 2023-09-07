## [N-01] Lack of address(0) checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

There are 4 instances of this issue in 4 files:

    File: contracts/bridge/SourceBridge.sol	

    40: constructor(
    41:   address _token,
    42:   address _axelarGateway,
    43:   address _gasService,
    44:   address owner
    45: ) {

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol

    File: contracts/bridge/DestinationBridge.sol	

    55: constructor(
    56:   address _token,
    57:   address _axelarGateway,
    58:   address _allowlist,
    59:   address _ondoApprover,
    60:   address _owner,
    61:   uint256 _mintLimit,
    62:   uint256 _mintDuration
    63: )

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

    File: contracts/usdy/rUSDYFactory.sol	

    51: constructor(address _guardian) {
    52:   guardian = _guardian;
    53: }

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDYFactory.sol

    File: contracts/rwaOracles/RWADynamicOracle.sol	

    30: constructor(
    31:   address admin,
    32:   address setter,
    33:   address pauser,
    34:   uint256 firstRangeStart,
    35:   uint256 firstRangeEnd,
    36:   uint256 dailyIR,
    37:   uint256 startPrice
    38: ) {

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

## [N-02] According to the syntax rules, use `=> mapping (` instead of `=> mapping(` using spaces as keyword

There are 2 instances of this issue in 2 files:

    File: contracts/bridge/DestinationBridge.sol	

    45: mapping(bytes32 => mapping(uint256 => bool)) public isSpentNonce;

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/DestinationBridge.sol

    File: contracts/usdy/rUSDY.sol	

    79: mapping(address => mapping(address => uint256)) private allowances;

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/usdy/rUSDY.sol

## [N-03] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

There is 1 instance of this issue in 1 file:

    File: contracts/rwaOracles/RWADynamicOracle.sol	

    350: assembly {

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/rwaOracles/RWADynamicOracle.sol

## [N-04] File does not contain an SPDX Identifier

There is 1 instance of this issue in 1 file:

    File: contracts/bridge/SourceBridge.sol	

    1: pragma solidity 0.8.16;

https://github.com/code-423n4/2023-09-ondo/blob/main/contracts/bridge/SourceBridge.sol
    
## [N-05] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

1. splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
2. clearly documenting the functions and implementations the owner can change,
3. documenting the risks associated with privileged users and single points of failure, and
4. ensuring that users are aware of all the risks associated with the system.

## [N-06] Empty/Unused Function Parameters

Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages.

There is 1 instance of this issue in 1 file:

As an example, the following function may have these parameters refactored to:

```
626: function _beforeTokenTransfer(
627:   address from,
628:   address to,
629:   uint256
630: ) internal view {
```
    function _beforeTokenTransfer(
      address from,
      address to,
      uint256 /* extradata */
    ) internal view {