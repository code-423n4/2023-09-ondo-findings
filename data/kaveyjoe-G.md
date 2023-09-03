## [G-01] - Gas Inefficiency in burnAndCallAxelar

**Description:** The burnAndCallAxelar function has some gas inefficiencies, especially with multiple state changes and function calls within it.

**Code snippet:**


TOKEN.burnFrom(msg.sender, amount);

bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

_payGasAndCallContract(destinationChain, destContract, payload);

**Recommendation**
 Combine the state changes into a single external call, which can save gas by reducing the number of storage writes.


function burnAndCallAxelar(
    uint256 amount,
    string calldata destinationChain
) external payable whenNotPaused {
    // ... (Check destinationChain and msg.value)

    // Encode payload
    bytes memory payload = abi.encode(VERSION, msg.sender, amount, nonce++);

    // Perform token transfer and contract call in a single external call
    TOKEN.burnFromAndCallAxelar(
        msg.sender,
        amount,
        destinationChain,
        destContract,
        payload,
        msg.value
    );
}

## [G-02] - Gas Efficiency in _payGasAndCallContract

**Description:** The _payGasAndCallContract function performs two separate external calls (GAS_RECEIVER.payNativeGasForContractCall and AXELAR_GATEWAY.callContract) with separate gas costs. It can be optimized to save gas.

**Code snippet:**


GAS_RECEIVER.payNativeGasForContractCall{value: msg.value}(
    address(this),
    destinationChain,
    destContract,
    payload,
    msg.sender
);

AXELAR_GATEWAY.callContract(destinationChain, destContract, payload);

**Recommendation change :** Combine both external calls into a single external call to reduce gas costs.


function _payGasAndCallContract(
    string calldata destinationChain,
    string memory destContract,
    bytes memory payload
) private {
    address contractAddress = AXELAR_GATEWAY.getContractAddress(destinationChain, destContract);
    require(contractAddress != address(0), "Invalid contract address");

    (bool success, ) = contractAddress.call{value: msg.value}(payload);
    require(success, "External call failed");
}
## [G-03] - Avoid Redundant Storage Reads

**Description:** In the burnAndCallAxelar function, you read destContract from storage, but then you immediately check its length. This can be optimized to reduce gas consumption by avoiding unnecessary storage reads.

**Code snippet:**


string memory destContract = destChainToContractAddr[destinationChain];

if (bytes(destContract).length == 0) {
    revert DestinationNotSupported();
}

**Recommended change:** Optimize by checking the length directly without assigning destContract.


if (bytes(destChainToContractAddr[destinationChain]).length == 0) {
    revert DestinationNotSupported();
}

## [G-04] - Gas Efficiency in _execute

**Description:** The _execute function has multiple gas inefficiencies, including multiple storage reads and checks. We can optimize it to reduce gas consumption.

**Code snippet:**


(bytes32 version, address srcSender, uint256 amt, uint256 nonce) = abi.decode(payload, (bytes32, address, uint256, uint256));

if (version != VERSION) {
  revert InvalidVersion();
}

if (chainToApprovedSender[srcChain] == bytes32(0)) {
  revert ChainNotSupported();
}

if (chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr))) {
  revert SourceNotSupported();
}

if (isSpentNonce[chainToApprovedSender[srcChain]][nonce]) {
  revert NonceSpent();
}

**Recommended Change:** Optimize by reducing the number of storage reads and checks.


(bytes32 version, address srcSender, uint256 amt, uint256 nonce) = abi.decode(payload, (bytes32, address, uint256, uint256));

if (version != VERSION || chainToApprovedSender[srcChain] == bytes32(0) || chainToApprovedSender[srcChain] != keccak256(abi.encode(srcAddr)) || isSpentNonce[chainToApprovedSender[srcChain]][nonce]) {
  revert InvalidTransaction();
}

## [G-05] - Avoid Redundant Storage Reads

**Description:** In the _attachThreshold function, you read thresholds from storage and iterate through it multiple times. we can optimize this to reduce gas consumption by avoiding unnecessary storage reads.

**Code snippet:**


Threshold[] memory thresholds = chainToThresholds[srcChain];
for (uint256 i = 0; i < thresholds.length; ++i) {
  Threshold memory t = thresholds[i];
  // ...
}

**Recommended Change:** Optimize by reading thresholds once and using it directly.


Threshold[] memory thresholds = chainToThresholds[srcChain];
for (uint256 i = 0; i < thresholds.length; ++i) {
  Threshold memory t = thresholds[i];
  // ...
}

## [G-06] - Gas Efficiency in _mintIfThresholdMet

**Description:** The _mintIfThresholdMet function has some gas inefficiencies. we can optimize it by using memory variables and avoiding multiple storage reads.

**Code snippet:**


bool thresholdMet = _checkThresholdMet(txnHash);
Transaction memory txn = txnHashToTransaction[txnHash];
if (thresholdMet) {
  // ...
}

**Recommended Change:** Optimize by using memory variables for txnHashToTransaction[txnHash] and txnToThresholdSet[txnHash].


Transaction memory txn;
TxnThreshold memory t;
(t.approvers, t.numberOfApprovalsNeeded) = txnToThresholdSet[txnHash];
(txn.sender, txn.amount) = (txnHashToTransaction[txnHash].sender, txnHashToTransaction[txnHash].amount);

bool thresholdMet = _checkThresholdMet(txnHash);
if (thresholdMet) {
  // ...
}

## [G-07] - Inefficient Variable Assignments

**Location:** Inside the _mintShares function.

**Description:** Gas can be optimized by eliminating unnecessary variable assignments.

**Code snippet:**


uint256 currentSenderShares = shares[_sender];
totalShares += _sharesAmount;
shares[_recipient] = shares[_recipient] + _sharesAmount;

**Recommended Change:**


totalShares += _sharesAmount;
shares[_sender] -= _sharesAmount;
shares[_recipient] += _sharesAmount;

## [G-08] - Redundant Variable Declaration

**Location:** Inside the _burnShares function.

**Description:** A redundant variable declaration can be avoided to save gas.

**Code snippet:**


uint256 preRebaseTokenAmount = getRUSDYByShares(_sharesAmount);
uint256 postRebaseTokenAmount = getRUSDYByShares(_sharesAmount);

**Recommended Change:**


uint256 tokenAmount = getRUSDYByShares(_sharesAmount);

## [G-09] - Unnecessary Division and Multiplication

**Location:** Inside the balanceOf and getRUSDYByShares functions.

**Description:** Gas can be saved by eliminating redundant division and multiplication operations.

**Code snippet:**


return (_sharesOf(_account) * oracle.getPrice()) / (1e18 * BPS_DENOMINATOR);

**Recommended Change:**


return (_sharesOf(_account) * oracle.getPrice()) / 1e18;

## [G-10] - Redundant Checks

**Location:** Inside the _beforeTokenTransfer function.

**Description:** Redundant checks can be avoided by reusing the result of _isBlocked, _isSanctioned, and _isAllowed.

**Code snippet:**


require(!_isBlocked(msg.sender), "rUSDY: 'sender' address blocked");
require(!_isSanctioned(msg.sender), "rUSDY: 'sender' address sanctioned");
require(_isAllowed(msg.sender), "rUSDY: 'sender' address not on allowlist");

**Recommended Change:**


bool isBlocked = _isBlocked(msg.sender);
bool isSanctioned = _isSanctioned(msg.sender);
bool isAllowed = _isAllowed(msg.sender);

require(!isBlocked, "rUSDY: 'sender' address blocked");
require(!isSanctioned, "rUSDY: 'sender' address sanctioned");
require(isAllowed, "rUSDY: 'sender' address not on allowlist");

## [G-11] - Redundant Event Emissions

**Location:** Inside the _mintShares and _burnShares functions.

**Description:** The Transfer event is already emitted in the _transferShares function; additional emissions can be avoided to save gas.

**Code snippet:**


emit Transfer(address(0), msg.sender, getRUSDYByShares(_USDYAmount));
emit TransferShares(address(0), msg.sender, _USDYAmount);

**Recommendation**
 Remove the redundant event emissions.