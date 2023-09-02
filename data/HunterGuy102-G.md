# Use named imports instead of plain `import file.sol

#### Using import declarations of the form import {<identifier_name>} from 'some/file.sol' avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

``` Solidity
File: SourceBridge.sol

3: import "contracts/interfaces/IAxelarGateway.sol";
4: import "contracts/interfaces/IAxelarGasService.sol";
5: import "contracts/interfaces/IMulticall.sol";
6: import "contracts/interfaces/IRWALike.sol";
8: import "contracts/external/openzeppelin/contracts/access/Ownable.sol";
9: import "contracts/external/openzeppelin/contracts/security/Pausable.sol";

File: DestinationBridge.sol

18: import "contracts/interfaces/IAxelarGateway.sol";
19: import "contracts/interfaces/IAxelarGasService.sol";
20: import "contracts/external/axelar/AxelarExecutable.sol";
21: import "contracts/interfaces/IRWALike.sol";
22: import "contracts/interfaces/IAllowlist.sol";
23: import "contracts/external/openzeppelin/contracts/access/Ownable.sol";
24: import "contracts/external/openzeppelin/contracts/security/Pausable.sol";
25: import "contracts/bridge/MintRateLimiter.sol";

File: rUSDY.sol

18: import "contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
19: import "contracts/external/openzeppelin/contracts-upgradeable/token/ERC20/IERC20MetadataUpgradeable.sol";
20: import "contracts/external/openzeppelin/contracts-upgradeable/proxy/Initializable.sol";
21: import "contracts/external/openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol";
22: import "contracts/external/openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
23: import "contracts/external/openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
24: import "contracts/usdy/blocklist/BlocklistClientUpgradeable.sol";
25: import "contracts/usdy/allowlist/AllowlistClientUpgradeable.sol";
26: import "contracts/sanctions/SanctionsListClientUpgradeable.sol";
27: import "contracts/interfaces/IUSDY.sol";
28: import "contracts/rwaOracles/IRWADynamicOracle.sol";

File: rUSDYFactory.sol

19: import "contracts/external/openzeppelin/contracts/proxy/ProxyAdmin.sol";
20: import "contracts/Proxy.sol";
21: import "contracts/usdy/rUSDY.sol";
22: import "contracts/interfaces/IMulticall.sol";

File: RWADynamicOracle.sol

18: import "contracts/rwaOracles/IRWAOracle.sol";
19: import "contracts/external/openzeppelin/contracts/access/AccessControlEnumerable.sol";
20: import "contracts/external/openzeppelin/contracts/security/Pausable.sol";
```