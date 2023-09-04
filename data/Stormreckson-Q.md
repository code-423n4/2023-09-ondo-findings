1- The following functions in `rUSDY.sol` lack checks on the returned values of the external calls made.
Not checking the return value of an external call can result in undesired state changes within the contract. This may lead to loss of funds, incorrect calculations or balances, or unintended consequences that can negatively impact the contract and its users.
Adding return value checks can be beneficial to ensure that the intended actions are executed successfully and to handle any potential errors or issues that may arise during the execution of the function. 

`Burn()`

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L680-L681

```
    usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR);
```
`Unwrap()`

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L454-L455

```
usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR);
```


https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L436

```
    usdy.transferFrom(msg.sender, address(this), _USDYAmount);
```
`Wrap()`

example of how you can add return value checks to the transfer function calls in the unwrap and burn functions:

i. `Unwrap` Function:

```
function unwrap(uint256 _rUSDYAmount) external whenNotPaused {
  require(_rUSDYAmount > 0, "rUSDY: can't unwrap zero rUSDY tokens");
  uint256 usdyAmount = getSharesByRUSDY(_rUSDYAmount);
  if (usdyAmount < BPS_DENOMINATOR) revert UnwrapTooSmall();
  _burnShares(msg.sender, usdyAmount);
  require(usdy.transfer(msg.sender, usdyAmount / BPS_DENOMINATOR), "Transfer failed");
  emit TokensBurnt(msg.sender, _rUSDYAmount);
}
```

ii. `Burn` Function:

```
function burn(address _account, uint256 _amount) external onlyRole(BURNER_ROLE) {
  uint256 sharesAmount = getSharesByRUSDY(_amount);
  _burnShares(_account, sharesAmount);
  require(usdy.transfer(msg.sender, sharesAmount / BPS_DENOMINATOR), "Transfer failed");
  emit TokensBurnt(_account, _amount);
}
```

In the modified code, the require` statement checks if the transfer of USDY tokens was successful. If the transfer fails, the code reverts with an error message.


...
2- #Tittle 
variable shadowing issue

Variable shadowing occurs when a function parameter or variable has the same name as a higher-level scope variable. In this case, the function parameters blocklist, allowlist, and sanctionsList have the same names as the state variables that are being set in the respective functions.

While Solidity allows using the same names for function parameters and state variables, it can lead to confusion and unintended behavior. In this case, the function parameters will shadow the state variables, making it unclear which variable is being referenced within the function.

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L693-L726

```
  function setBlocklist(
    address blocklist
  ) external override onlyRole(LIST_CONFIGURER_ROLE) {
    _setBlocklist(blocklist);
  }
```

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L698-L703)

```
  function setAllowlist(
    address allowlist
  ) external override onlyRole(LIST_CONFIGURER_ROLE) {
    _setAllowlist(allowlist);
  }
```

(https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L709-L715)

```
  function setSanctionsList(
    address sanctionsList
  ) external override onlyRole(LIST_CONFIGURER_ROLE) {
    _setSanctionsList(sanctionsList);
  }
```
(https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDY.sol#L720-L724)


To avoid this issue, you can consider using a naming convention or prefixing the function parameters to differentiate them from the state variables. For example, you can prefix the function parameters with an underscore or use a different naming convention, like `newBlocklist`, `newAllowlist`, and `newSanctionsList`.

Here's an example of how you can update the code to avoid variable shadowing:
```
function setBlocklist(
  address _blocklist
) external override onlyRole(LIST_CONFIGURER_ROLE) {
  _setBlocklist(_blocklist);
}

function setAllowlist(
  address _allowlist
) external override onlyRole(LIST_CONFIGURER_ROLE) {
  _setAllowlist(_allowlist);
}

function setSanctionsList(
  address _sanctionsList
) external override onlyRole(LIST_CONFIGURER_ROLE) {
  _setSanctionsList(_sanctionsList);
}
```
By using distinct names for the function parameters, you can avoid potential confusion.


.....
3-https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L197-L204

```
   function approve(bytes32 txnHash) external {
    if (!approvers[msg.sender]) {
      revert NotApprover();
    }
    _approve(txnHash);
    _mintIfThresholdMet(txnHash);
  }
  ```
It's important to validate the `txnHash` parameter in the `approve` function before further processing. Ensure that it is not an empty or zero value, as it could lead to unexpected behavior. You can add a require statement at the beginning of the `approve` function to check if `txnHash` is valid.

 Fix:
```  
 require(txnHash != 0, "Invalid txnHash");
```

....
4-https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L157

#Issue
Event Logging:

 It's recommended to emit appropriate events to log the approval status and other relevant information. This helps in transparency and auditing. Consider emitting events after successful approvals.

   Example:
```
   event Approval(address indexed approver, bytes32 indexed txnHash);
   

   function _approve(bytes32 txnHash) internal {
       // Approval logic
       emit Approval(msg.sender, txnHash);
   }
```
  
   
.....

5-https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/DestinationBridge.sol#L61-L66

The current implementation of the constructor function lacks validation to check of _`mintLimit` and `_mintDuration` are zero
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

it would be advisable to include a check to ensure that `mintLimit` and `mintDuration` are not zero in the constructor function. This check can help prevent potential issues or unintended behavior in the contract.

Here's an updated version of the constructor function that includes the check:

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
    require(_mintLimit > 0, "Mint limit must be greater than zero");
    require(_mintDuration > 0, "Mint duration must be greater than zero");

    TOKEN = IRWALike(_token);
    AXELAR_GATEWAY = IAxelarGateway(_axelarGateway);
    ALLOWLIST = IAllowlist(_allowlist);
    approvers[_ondoApprover] = true;
    _transferOwnership(_owner);
  }
```

By adding the `require` statements, the constructor will check if `mintLimit` and `mintDuration` are greater than zero. If either of them is zero, the constructor will revert the transaction and provide an appropriate error message.

Including this check enhances the robustness and reliability of the contract by ensuring that the provided values for `mintLimit` and `mintDuration` are valid and within the expected range.



..
6-
https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L131-L135


https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L165-L169

Title: Missing Checks in multiexcall Function

This issue is also present in `SourceBridge.sol` in the `multiexcall` function 

https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/bridge/SourceBridge.sol#L165-L169

The `multiexcall` function allows for arbitrary batched calls to external contracts. However, it was identified that several important checks and considerations are missing. 

```
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


i. Event Logging:
The code lacks event logging, which is essential for transparency and auditing purposes. Emitting events allows for proper tracking and analysis of the function's execution. Consider adding events to capture important information, such as the target contract, the executed data, the value, and the result of each individual call. This will enhance the visibility and traceability of the function's actions.

Recommendation:
- Implement event logging to provide transparency and auditing capabilities.
- Emit events to capture relevant information related to the executed calls.

ii. Error Handling:
Although the code checks the success of each external call with `require(success, "Call Failed")`, it lacks detailed information about the cause of the failure. This can make debugging and troubleshooting challenging. It is advisable to include additional error messages or codes to provide more specific feedback on the reason for the failure. This will aid in identifying and resolving potential issues more efficiently.

Recommendation:
- Enhance the error handling mechanism by providing detailed error messages or codes.
- Include information about the cause of the failure to facilitate debugging and troubleshooting.

iii. Input Validation:
The code does not explicitly validate the input parameters in the `exCallData` array. Without proper input validation, the function may be susceptible to vulnerabilities or misuse. It is crucial to implement appropriate input validation checks to ensure the integrity and validity of the data provided within each `ExCallData` struct. Consider validating the array to ensure it is not empty and verifying the integrity of the data before executing the calls.

Recommendation:
- Implement input validation checks on the `exCallData` array.
- Verify the integrity and validity of the data within each `ExCallData` struct.



...
7-https://github.com/code-423n4/2023-09-ondo/blob/47d34d6d4a5303af5f46e907ac2292e6a7745f6c/contracts/usdy/rUSDYFactory.sol#L75-L76

Title: Implement Contract Tracking Mechanism for deployed `rUSDY` Contract

implementing a mechanism to track deployed contracts in the `rUSDYFactory.sol` . This addition will enhance contract management comprehensively and improve overall efficiency.

 deploying contracts without tracking their addresses, resulting in a lack of visibility and potential challenges in managing deployed instances. Without a tracking mechanism, it becomes difficult to keep an organized record of deployed contracts and access them efficiently.

Proposed Solution:
   It is recommend to implement a contract tracking mechanism by introducing an array (or mapping) variable to store the addresses of deployed contracts. This solution will offer the following comprehensive benefits:

   a. Enhanced Contract Management:
      By tracking deployed contracts, the development team and stakeholders can maintain a centralized record of all contract instances. This record provides a comprehensive overview of the deployed contracts, facilitating easier access, monitoring, and management.

   b. Improved Contract Interaction:
      Having a tracking mechanism enables efficient interactions with deployed contracts. The recorded addresses can be used to quickly reference specific contracts, enabling seamless integration with other systems, dApps, or smart contracts that require interaction with the deployed instances.

   c. Simplified Auditing and Compliance:
      Keeping a comprehensive record of deployed contracts supports auditing and compliance processes. It allows auditors and compliance officers to easily verify the existence and integrity of deployed contracts, ensuring adherence to regulatory requirements and internal policies.

   d. Future Contract Upgrades and Maintenance:
      The contract tracking mechanism becomes invaluable when planning upgrades or maintenance activities. It provides a clear list of deployed contracts, making it easier to identify which contracts require updates or modifications. This assists in streamlining the upgrade process and minimizing the risk of overlooking any deployed instances.

Implementation Approach:
   To implement the contract tracking mechanism, it is suggested  the following steps:

   a. Declare an array variable:
      Introduce an array variable, such as `deployedrUSDYContracts`, to store the addresses of deployed contracts.

   b. Update the deployment process:
      After deploying each contract, push the contract's address into the `deployedrUSDYContracts` array.

   c. Utilize the tracking mechanism:
      Access the `deployedrUSDYContracts`array whenever there is a need to retrieve or manage the list of deployed contract addresses.

Conclusion:
   Implementing a contract tracking mechanism through the suggested approach will greatly contribute to comprehensive contract management. It enhances visibility, simplifies auditing, and enables smooth interactions with deployed contracts. By adopting this best practice, the development team can streamline contract maintenance, promote efficient upgrades, and ensure compliance with regulatory requirements.
