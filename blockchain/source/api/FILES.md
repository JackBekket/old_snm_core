# blockchain/source/api/AddressHashMap.go  
## Package: api  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (ethereum core functionality)  
*   `github.com/ethereum/go-ethereum/accounts/abi` (ABI encoding/decoding)  
*   `github.com/ethereum/go-ethereum/accounts/abi/bind` (Contract binding utilities)  
*   `github.com/ethereum/go-ethereum/common` (Common Ethereum types like addresses)  
*   `github.com/ethereum/go-ethereum/core/types` (Ethereum transaction and block types)  
*   `github.com/ethereum/go-ethereum/event` (Event filtering and subscription)  
  
**External Data/Input Sources:**  
  
*   `AddressHashMapABI`: JSON ABI definition of the smart contract. This is used for interacting with the contract.  
*   `AddressHashMapBin`: Compiled bytecode of the smart contract. This is used for deploying the contract.  
*   `common.Address`: Ethereum addresses used for contract deployment, function calls, and event filtering.  
*   `bind.TransactOpts`: Transaction options (e.g., gas limit, gas price, nonce, from address) for sending transactions to the contract.  
*   `bind.CallOpts`: Call options for reading data from the contract.  
*   `ethereum.Subscription`: Used for watching events emitted by the contract.  
  
**TODOs:**  
  
There are no explicit `TODO` comments in the provided code.  
  
### Code Summary  
  
**1. Contract Binding Generation:**  
  
The code is auto-generated from a smart contract ABI and bytecode. It provides Go bindings for interacting with an Ethereum smart contract named `AddressHashMap`. The bindings include:  
  
*   `DeployAddressHashMap`: Function to deploy a new instance of the contract.  
*   `AddressHashMap`: Struct containing caller, transactor, and filterer interfaces for interacting with the contract.  
*   `AddressHashMapCaller`: Read-only interface for calling constant functions.  
*   `AddressHashMapTransactor`: Write-only interface for sending transactions.  
*   `AddressHashMapFilterer`: Interface for filtering and subscribing to contract events.  
  
**2. Core Functionality:**  
  
The contract exposes the following functions:  
  
*   `Owner()`: Returns the contract owner's address.  
*   `Read(key [32]byte)`: Reads an address value associated with a given key.  
*   `Write(key [32]byte, value common.Address)`: Writes an address value to a given key.  
*   `RenounceOwnership()`: Renounces ownership of the contract.  
*   `TransferOwnership(newOwner common.Address)`: Transfers ownership to a new address.  
  
**3. Event Handling:**  
  
The contract emits two events:  
  
*   `OwnershipRenounced(previousOwner address)`: Emitted when ownership is renounced.  
*   `OwnershipTransferred(previousOwner address, newOwner address)`: Emitted when ownership is transferred.  
  
The code provides functions for filtering and watching these events using `FilterOwnershipRenounced`, `WatchOwnershipRenounced`, `FilterOwnershipTransferred`, and `WatchOwnershipTransferred`.  
  
**4. Session Management:**  
  
The code includes session types (`AddressHashMapSession`, `AddressHashMapCallerSession`, `AddressHashMapTransactorSession`) to pre-configure call and transaction options for easier use.  
  
**5. Raw Access:**  
  
The `AddressHashMapRaw`, `AddressHashMapCallerRaw`, and `AddressHashMapTransactorRaw` types provide low-level access to the contract's underlying methods.  
  
# blockchain/source/api/BasicToken.go  
## Package: api  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including sub-packages: `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `BasicTokenABI`: A JSON string representing the Application Binary Interface (ABI) of the BasicToken contract. This is used for interacting with the contract.  
*   `BasicTokenBin`: A hexadecimal string representing the compiled bytecode of the BasicToken contract. This is used for deploying the contract.  
*   Ethereum address (`common.Address`) for contract deployment and interaction.  
*   Transaction options (`bind.TransactOpts`) for signing and sending transactions.  
*   Call options (`bind.CallOpts`) for reading contract state.  
*   Filter options (`bind.FilterOpts`) for event filtering.  
  
**TODOs:**  
  
*   No TODO comments found in the provided code.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
The code provides auto-generated Go bindings for interacting with an Ethereum smart contract named `BasicToken`. It uses the `github.com/ethereum/go-ethereum` library to handle contract deployment, method calls, and event filtering. The bindings are generated from the `BasicTokenABI` and `BasicTokenBin` constants.  
  
### Contract Structures  
  
The code defines several structures:  
  
*   `BasicToken`: The main binding structure, providing access to caller, transactor, and filterer interfaces.  
*   `BasicTokenCaller`: Read-only interface for calling constant functions.  
*   `BasicTokenTransactor`: Write-only interface for sending transactions.  
*   `BasicTokenFilterer`: Interface for filtering and listening to contract events.  
*   Session types (`BasicTokenSession`, `BasicTokenCallerSession`, `BasicTokenTransactorSession`) for pre-configured call and transaction options.  
*   Raw types (`BasicTokenRaw`, `BasicTokenCallerRaw`, `BasicTokenTransactorRaw`) for low-level access.  
  
### Deployment Function  
  
The `DeployBasicToken` function deploys a new `BasicToken` contract to the Ethereum blockchain. It takes transaction options and a backend as input, compiles the bytecode, and returns the contract address, transaction hash, and a bound `BasicToken` instance.  
  
### Contract Methods  
  
The code includes bindings for the following contract methods:  
  
*   `balanceOf`: Returns the token balance of an address.  
*   `totalSupply`: Returns the total token supply.  
*   `transfer`: Transfers tokens to another address.  
  
### Event Filtering  
  
The code provides functionality for filtering and watching the `Transfer` event emitted by the `BasicToken` contract. The `FilterTransfer` and `WatchTransfer` functions allow retrieving past events or subscribing to new events in real-time.  
  
# blockchain/source/api/Blacklist.go  
## Package: api  
  
This file contains auto-generated bindings for the `Blacklist` Ethereum contract. It's designed to interact with a deployed smart contract on the Ethereum blockchain.  
  
**Imports:**  
  
*   `strings`: For string manipulation (used in ABI parsing).  
*   `github.com/ethereum/go-ethereum`: Core Ethereum library for interacting with the blockchain.  
*   `github.com/ethereum/go-ethereum/accounts/abi`: For ABI (Application Binary Interface) parsing.  
*   `github.com/ethereum/go-ethereum/accounts/abi/bind`: For binding to smart contracts.  
*   `github.com/ethereum/go-ethereum/common`: For Ethereum address and other common types.  
*   `github.com/ethereum/go-ethereum/core/types`: For transaction and block types.  
*   `github.com/ethereum/go-ethereum/event`: For event filtering.  
  
**External Data/Input Sources:**  
  
*   `BlacklistABI`: A JSON string representing the contract's ABI, used for decoding function calls and events.  
*   `BlacklistBin`: The compiled bytecode of the contract, used for deployment.  
*   Ethereum blockchain: The contract interacts with the Ethereum blockchain for transactions and state.  
  
**TODOs:**  
  
There are no explicit `TODO` comments in the provided code.  
  
### Code Summary:  
  
**1. Contract Binding Generation:**  
  
The code generates Go bindings for the `Blacklist` contract, allowing developers to interact with it programmatically. It includes functions for deploying the contract (`DeployBlacklist`) and creating different types of bindings: read-only (`BlacklistCaller`), write-only (`BlacklistTransactor`), and event filtering (`BlacklistFilterer`).  
  
**2. Contract Structures:**  
  
Several structs are defined to represent different aspects of the contract interaction: `Blacklist`, `BlacklistCaller`, `BlacklistTransactor`, `BlacklistFilterer`, `BlacklistSession`, etc. These structures provide methods for calling functions, sending transactions, and filtering events.  
  
**3. Function Bindings:**  
  
The code includes bindings for all the contract's functions, such as `Check`, `Market`, `Owner`, `Add`, `Remove`, `AddMaster`, `RemoveMaster`, `SetMarketAddress`, `RenounceOwnership`, and `TransferOwnership`. Each function has corresponding methods in the generated bindings.  
  
**4. Event Filtering:**  
  
The code provides mechanisms for filtering events emitted by the contract, such as `AddedToBlacklist`, `RemovedFromBlacklist`, `OwnershipRenounced`, and `OwnershipTransferred`. These filters allow developers to monitor specific events on the blockchain.  
  
**5. Low-Level Access:**  
  
The `BlacklistRaw`, `BlacklistCallerRaw`, and `BlacklistTransactorRaw` types provide low-level access to the contract's methods, bypassing the higher-level abstractions.  
  
**6. Deployment:**  
  
The `DeployBlacklist` function handles the deployment of the contract to the Ethereum blockchain.  
  
# blockchain/source/api/DeployList.go  
## Package: api  
  
**Imports:**  
  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (ethereum)  
*   `github.com/ethereum/go-ethereum/accounts/abi`  
*   `github.com/ethereum/go-ethereum/accounts/abi/bind`  
*   `github.com/ethereum/go-ethereum/common`  
*   `github.com/ethereum/go-ethereum/core/types`  
*   `github.com/ethereum/go-ethereum/event`  
  
**External Data/Input Sources:**  
  
*   `DeployListABI`: A JSON string representing the Application Binary Interface (ABI) of the `DeployList` contract. This is used for interacting with the contract.  
*   `DeployListBin`: A hexadecimal string representing the compiled bytecode of the `DeployList` contract. This is used for deploying the contract.  
*   `_deployers`: A slice of Ethereum addresses (`common.Address`) used as input during contract deployment.  
*   Ethereum blockchain: The code interacts with the Ethereum blockchain through `bind.ContractBackend` and `bind.TransactOpts`.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
This section defines the `DeployList` contract binding, generated from the provided ABI (`DeployListABI`) and bytecode (`DeployListBin`). The code includes structures for interacting with the contract in read-only (`DeployListCaller`), write-only (`DeployListTransactor`), and event filtering (`DeployListFilterer`) modes.  It also provides session types (`DeployListSession`, `DeployListCallerSession`, `DeployListTransactorSession`) for managing call and transaction options. Raw access types (`DeployListRaw`, `DeployListCallerRaw`, `DeployListTransactorRaw`) are also defined for low-level interaction.  
  
### Deployment Function  
  
The `DeployDeployList` function deploys a new instance of the `DeployList` contract to the Ethereum blockchain. It takes transaction options (`auth`), a backend (`backend`), and a list of deployers (`_deployers`) as input. It returns the contract address, transaction details, and a bound contract instance.  
  
### Contract Methods  
  
The code defines methods for interacting with the `DeployList` contract, including:  
  
*   `GetDeployers`: Retrieves a list of deployers.  
*   `Owner`: Retrieves the contract owner.  
*   `AddDeployer`: Adds a new deployer.  
*   `RemoveDeployer`: Removes an existing deployer.  
*   `RenounceOwnership`: Renounces ownership of the contract.  
*   `TransferOwnership`: Transfers ownership of the contract to a new address.  
  
### Event Filtering  
  
The code includes structures and functions for filtering events emitted by the `DeployList` contract:  
  
*   `DeployerAdded`: Filters for events when a deployer is added.  
*   `DeployerRemoved`: Filters for events when a deployer is removed.  
*   `OwnershipRenounced`: Filters for events when ownership is renounced.  
*   `OwnershipTransferred`: Filters for events when ownership is transferred.  
  
The filtering mechanisms provide iterators (`DeployListDeployerAddedIterator`, `DeployListDeployerRemovedIterator`, `DeployListOwnershipRenouncedIterator`, `DeployListOwnershipTransferredIterator`) for processing events and watchers (`WatchDeployerAdded`, `WatchDeployerRemoved`, `WatchOwnershipRenounced`, `WatchOwnershipTransferred`) for real-time event monitoring.  
  
# blockchain/source/api/DevicesStorage.go  
```  
Package: api  
  
Imports:  
- math/big  
- strings  
- github.com/ethereum/go-ethereum (including subpackages: accounts/abi, accounts/abi/bind, common, core/types, event)  
  
External Data/Input Sources:  
- ABI (Application Binary Interface) for the contract, stored as a string constant `DevicesStorageABI`.  
- Bytecode for the contract, stored as a string constant `DevicesStorageBin`.  
- Ethereum address for contract deployment or interaction.  
- Ethereum transaction options (`bind.TransactOpts`) for contract interactions.  
- Ethereum contract backend (`bind.ContractBackend`) for deployment and calls.  
  
TODOs:  
- None found in the provided code snippet.  
  
Summary:  
  
The code defines a Go binding for an Ethereum smart contract named `DevicesStorage`. It's generated and should not be manually edited. The binding provides functions for deploying the contract (`DeployDevicesStorage`), interacting with its functions (e.g., `GetDevices`, `Hash`, `KillDevicesStorage`, `SetDevices`, `Touch`, `TransferOwnership`, `RenounceOwnership`), and listening for events (`DevicesHasSet`, `DevicesTimestampUpdated`, `DevicesUpdated`, `OwnershipRenounced`, `OwnershipTransferred`, `Suicide`).  
  
The code includes structures for read-only (`DevicesStorageCaller`), write-only (`DevicesStorageTransactor`), and event filtering (`DevicesStorageFilterer`) access to the contract. Session types (`DevicesStorageSession`, `DevicesStorageCallerSession`, `DevicesStorageTransactorSession`) are also defined for managing call and transaction options. Raw access types (`DevicesStorageRaw`, `DevicesStorageCallerRaw`, `DevicesStorageTransactorRaw`) are provided for low-level interactions.  
  
The code also includes event iterators (`DevicesStorageDevicesHasSetIterator`, `DevicesStorageDevicesTimestampUpdatedIterator`, `DevicesStorageDevicesUpdatedIterator`, `DevicesStorageOwnershipRenouncedIterator`, `DevicesStorageOwnershipTransferredIterator`, `DevicesStorageSuicideIterator`) for filtering and processing contract events. The event filtering functions (`FilterDevicesHasSet`, `FilterDevicesTimestampUpdated`, `FilterDevicesUpdated`, `FilterOwnershipRenounced`, `FilterOwnershipTransferred`, `FilterSuicide`) and watching functions (`WatchDevicesHasSet`, `WatchDevicesTimestampUpdated`, `WatchDevicesUpdated`, `WatchOwnershipRenounced`, `WatchOwnershipTransferred`, `WatchSuicide`) are provided for event monitoring.  
  
<end_of_output>  
```  
  
# blockchain/source/api/ERC20.go  
## ERC20 Binding Package Summary  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `ERC20ABI`: A JSON string representing the Application Binary Interface (ABI) of an ERC20 token contract. This is used for interacting with the contract.  
*   `ERC20Bin`: A hexadecimal string representing the compiled bytecode of an ERC20 token contract. This is used for deploying new contracts.  
*   Ethereum blockchain backend (`bind.ContractBackend`) for interacting with deployed contracts.  
*   Transaction options (`bind.TransactOpts`) for signing and sending transactions.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
This section defines constants `ERC20ABI` and `ERC20Bin` which are used to generate bindings for an ERC20 token contract. The `DeployERC20` function deploys a new ERC20 contract to the Ethereum blockchain using the provided ABI and bytecode.  
  
### ERC20 Structs  
  
The code defines several structs (`ERC20`, `ERC20Caller`, `ERC20Transactor`, `ERC20Filterer`, `ERC20Session`, `ERC20CallerSession`, `ERC20TransactorSession`, `ERC20Raw`, `ERC20CallerRaw`, `ERC20TransactorRaw`) that provide different interfaces for interacting with an ERC20 contract. These structs allow for read-only access, write access, and event filtering.  
  
### Contract Interaction Functions  
  
Functions like `NewERC20`, `NewERC20Caller`, `NewERC20Transactor`, and `NewERC20Filterer` create instances of these structs, bound to a specific deployed contract address. The `bindERC20` function handles the underlying binding process.  
  
### Call, Transfer, and Transact Functions  
  
Generic `Call`, `Transfer`, and `Transact` functions are provided for raw contract interaction. These are used internally by the more specific functions.  
  
### ERC20 Specific Functions  
  
Functions like `Allowance`, `BalanceOf`, `TotalSupply`, `Approve`, `Transfer`, and `TransferFrom` provide access to the standard ERC20 token functions. These functions are exposed through the `ERC20Caller` and `ERC20Transactor` structs.  
  
### Event Filtering  
  
The code includes functionality for filtering and watching for `Approval` and `Transfer` events emitted by the ERC20 contract. The `FilterApproval`, `WatchApproval`, `FilterTransfer`, and `WatchTransfer` functions provide mechanisms for subscribing to these events. The `ERC20ApprovalIterator` and `ERC20TransferIterator` structs are used to iterate over the filtered events.  
  
# blockchain/source/api/ERC20Basic.go  
## Package: api  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `ERC20BasicABI`: A JSON string representing the Application Binary Interface (ABI) of an ERC20-compatible smart contract. This is used for interacting with the contract.  
*   `ERC20BasicBin`: A hexadecimal string representing the compiled bytecode of the ERC20 contract. This is used for deploying the contract.  
*   Ethereum blockchain backend (`bind.ContractBackend`): Required for interacting with the Ethereum network.  
*   Transaction options (`bind.TransactOpts`): Used for signing and sending transactions.  
*   Contract addresses (`common.Address`): Used to specify the deployed contract instance.  
  
**TODOs:**  
  
*   No TODO comments found in the provided code.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
This section defines constants `ERC20BasicABI` and `ERC20BasicBin` which hold the ABI and bytecode for a basic ERC20 token contract. The `DeployERC20Basic` function deploys a new instance of this contract to the Ethereum blockchain, using the provided ABI and bytecode. It returns the contract address, transaction details, and a bound contract instance.  
  
### Contract Structures  
  
The code defines several structures for interacting with the ERC20 contract:  
  
*   `ERC20Basic`: The main binding structure, combining caller, transactor, and filterer interfaces.  
*   `ERC20BasicCaller`: Provides read-only access to contract functions.  
*   `ERC20BasicTransactor`: Provides write access to contract functions.  
*   `ERC20BasicFilterer`: Enables filtering and monitoring of contract events.  
*   Session types (`ERC20BasicSession`, `ERC20BasicCallerSession`, `ERC20BasicTransactorSession`) provide pre-configured call and transaction options.  
*   Raw types (`ERC20BasicRaw`, `ERC20BasicCallerRaw`, `ERC20BasicTransactorRaw`) expose low-level access to the contract.  
  
### Contract Function Bindings  
  
The code includes functions for creating new instances of the contract (`NewERC20Basic`, `NewERC20BasicCaller`, `NewERC20BasicTransactor`, `NewERC20BasicFilterer`) and a helper function `bindERC20Basic` to establish the connection.  
  
### Function Calls  
  
The code provides bindings for calling contract functions:  
  
*   `BalanceOf`: Retrieves the token balance of an address.  
*   `TotalSupply`: Retrieves the total token supply.  
*   `Transfer`: Transfers tokens to another address.  
  
### Event Filtering  
  
The code includes functionality for filtering and watching for `Transfer` events emitted by the contract:  
  
*   `FilterTransfer`: Retrieves past `Transfer` events.  
*   `WatchTransfer`: Subscribes to new `Transfer` events in real-time.  
*   `ERC20BasicTransferIterator`: Used to iterate over filtered events.  
*   `ERC20BasicTransfer`: Represents the structure of a `Transfer` event.  
  
# blockchain/source/api/Market.go  
```  
Package: api  
  
Imports:  
- "math/big"  
- "strings"  
- "github.com/ethereum/go-ethereum"  
- "github.com/ethereum/go-ethereum/accounts/abi"  
- "github.com/ethereum/go-ethereum/accounts/abi/bind"  
- "github.com/ethereum/go-ethereum/common"  
- "github.com/ethereum/go-ethereum/core/types"  
- "github.com/ethereum/go-ethereum/event"  
  
External Data/Input Sources:  
- Contract ABI (MarketABI)  
- Contract bytecode (MarketBin)  
- Ethereum address for contract deployment  
- Addresses for token, blacklist, oracle, profile registry  
- Benchmarks and netflags quantities (big.Int)  
- Transaction options (bind.TransactOpts, bind.CallOpts)  
- Event filter options (bind.FilterOpts, bind.WatchOpts)  
  
TODOs:  
- None explicitly present in the code.  
  
Summary:  
  
The `api` package provides Go bindings for interacting with a deployed Ethereum smart contract named `Market`. The contract appears to manage deals, orders, and worker registration.  
  
Key Components:  
- `Market`: The main struct providing access to contract functions.  
- `MarketCaller`: Read-only access to contract state.  
- `MarketTransactor`: Write access to contract functions.  
- `MarketFilterer`: For filtering and subscribing to contract events.  
- Deployment Function: `DeployMarket` deploys the contract.  
- Event Filters: Includes filters for `Billed`, `DealChangeRequestSet`, `DealChangeRequestUpdated`, `DealOpened`, `DealUpdated`, `OrderPlaced`, `OrderUpdated`, `OwnershipRenounced`, `OwnershipTransferred`, `Pause`, `Unpause`, `WorkerAnnounced`, `WorkerConfirmed`, `WorkerRemoved`, `NumBenchmarksUpdated`, and `NumNetflagsUpdated`.  
- Function Wrappers: Provides Go functions for calling contract methods (e.g., `GetBenchmarksQuantity`, `PlaceOrder`, `CancelOrder`, `OpenDeal`, `Bill`, `CreateChangeRequest`, `RegisterWorker`, `ConfirmWorker`, `RemoveWorker`, `SetBenchmarksQuantity`, `SetNetflagsQuantity`, `Pause`, `Unpause`, `TransferOwnership`).  
  
The code is auto-generated from the contract ABI and provides a type-safe interface for interacting with the deployed smart contract. The package includes comprehensive event filtering capabilities for monitoring contract activity.  
<end_of_output>  
```  
  
# blockchain/source/api/Migrations.go  
## Package: api  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `MigrationsABI`: A JSON string representing the Application Binary Interface (ABI) of the `Migrations` contract. This is used for interacting with the contract.  
*   `MigrationsBin`: A hexadecimal string representing the compiled bytecode of the `Migrations` contract. This is used for deploying the contract.  
*   Ethereum address for contract deployment and interaction.  
*   Transaction options (`bind.TransactOpts`) for signing and sending transactions.  
*   Call options (`bind.CallOpts`) for reading contract state.  
*   Filter options (`bind.FilterOpts`) for event filtering.  
  
**TODOs:**  
  
*   None explicitly present in the provided code snippet.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
This code generates Go bindings for an Ethereum smart contract named `Migrations`. The bindings allow developers to interact with the contract in a type-safe manner. The code includes constants for the ABI (`MigrationsABI`) and bytecode (`MigrationsBin`) of the contract.  
  
### Contract Structures  
  
The code defines several structures:  
  
*   `Migrations`: The main contract binding, providing access to both read-only and write-only methods.  
*   `MigrationsCaller`: Provides read-only access to contract functions.  
*   `MigrationsTransactor`: Provides write access to contract functions.  
*   `MigrationsFilterer`: Enables filtering and listening for contract events.  
*   Session types (`MigrationsSession`, `MigrationsCallerSession`, `MigrationsTransactorSession`) for pre-configured call/transact options.  
*   Raw types (`MigrationsRaw`, `MigrationsCallerRaw`, `MigrationsTransactorRaw`) for low-level access.  
  
### Deployment Function  
  
The `DeployMigrations` function deploys a new instance of the `Migrations` contract to the Ethereum blockchain. It takes transaction options and a backend as input and returns the contract address, transaction details, and a pointer to the `Migrations` binding.  
  
### Function Bindings  
  
The code includes generated functions for calling and transacting with the `Migrations` contract:  
  
*   `LastCompletedMigration()`: Reads the last completed migration number.  
*   `Owner()`: Reads the contract owner's address.  
*   `RenounceOwnership()`: Transfers ownership to the zero address.  
*   `SetCompleted()`: Sets the completed migration number.  
*   `TransferOwnership()`: Transfers ownership to a new address.  
*   `Upgrade()`: Upgrades the contract to a new implementation.  
  
### Event Filtering  
  
The code provides functions for filtering and watching for contract events:  
  
*   `FilterOwnershipRenounced()`: Filters for `OwnershipRenounced` events.  
*   `WatchOwnershipRenounced()`: Subscribes to `OwnershipRenounced` events.  
*   `FilterOwnershipTransferred()`: Filters for `OwnershipTransferred` events.  
*   `WatchOwnershipTransferred()`: Subscribes to `OwnershipTransferred` events.  
  
The event structures (`MigrationsOwnershipRenounced`, `MigrationsOwnershipTransferred`) contain the event data and the raw log.  
  
# blockchain/source/api/MultiSigWallet.go  
```  
Package: api  
  
Imports:  
- math/big  
- strings  
- github.com/ethereum/go-ethereum (multiple sub-packages used: core/types, accounts/abi, accounts/abi/bind, event)  
  
External Data/Input Sources:  
- Contract ABI (MultiSigWalletABI): A JSON string defining the contract's interface.  
- Contract Bin (MultiSigWalletBin): The compiled bytecode of the contract.  
- Ethereum backend (bind.ContractBackend): Used for interacting with the blockchain.  
- Transaction options (bind.TransactOpts): Used for signing and sending transactions.  
- Event filter options (bind.FilterOpts): Used for filtering events.  
- Addresses (common.Address): Used for identifying accounts and contracts.  
- Big integers (*big.Int): Used for representing large numbers (e.g., transaction values).  
- Byte arrays ([]byte): Used for representing arbitrary data.  
  
TODOs:  
- None found in the provided code.  
  
Summary:  
  
The code defines a Go binding for interacting with a MultiSigWallet Ethereum smart contract. It provides functions for deploying the contract (DeployMultiSigWallet) and creating instances of the contract for calling (NewMultiSigWalletCaller), transacting (NewMultiSigWalletTransactor), and filtering events (NewMultiSigWalletFilterer).  
  
The binding includes generated structs (MultiSigWallet, MultiSigWalletCaller, MultiSigWalletTransactor, MultiSigWalletFilterer) that wrap the underlying contract interaction logic.  It also defines session types (MultiSigWalletSession, etc.) for managing call and transaction options.  
  
The code includes numerous functions for calling contract methods (e.g., MAXOWNERCOUNT, Confirmations, GetTransactionCount) and sending transactions (e.g., AddOwner, ChangeRequirement, ConfirmTransaction).  It also provides event filtering capabilities for events like Confirmation, Deposit, Execution, ExecutionFailure, OwnerAddition, OwnerRemoval, and RequirementChange.  Iterators are provided for consuming event logs (e.g., MultiSigWalletConfirmationIterator).  
  
The code is auto-generated based on the provided ABI (MultiSigWalletABI) and bytecode (MultiSigWalletBin).  The generated code handles the low-level details of interacting with the Ethereum blockchain, allowing developers to interact with the contract in a type-safe and convenient manner.  
<end_of_output>  
```  
  
# blockchain/source/api/OracleUSD.go  
```markdown  
## Package: api  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `OracleUSDABI`: A JSON string representing the Application Binary Interface (ABI) of the `OracleUSD` contract. This is used for interacting with the contract.  
*   `OracleUSDBin`: A hexadecimal string representing the compiled bytecode of the `OracleUSD` contract. This is used for deploying new instances of the contract.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
### Code Summary  
  
**1. Contract Binding Generation:**  
  
This code is auto-generated and should not be manually edited. It provides Go bindings for interacting with an Ethereum smart contract named `OracleUSD`. The bindings include functions for deploying the contract (`DeployOracleUSD`), calling its methods (`GetCurrentPrice`, `Owner`), and sending transactions to modify its state (`RenounceOwnership`, `SetCurrentPrice`, `TransferOwnership`).  
  
**2. Struct Definitions:**  
  
Several structs are defined to facilitate interaction with the contract:  
  
*   `OracleUSD`: The main struct, combining read-only (`Caller`), write-only (`Transactor`), and event filtering (`Filterer`) capabilities.  
*   `OracleUSDCaller`, `OracleUSDTransactor`, `OracleUSDFilterer`: Structs providing specific access to read, write, and event-related functions, respectively.  
*   Session structs (`OracleUSDSession`, `OracleUSDCallerSession`, `OracleUSDTransactorSession`) for pre-configured call/transaction options.  
*   Raw structs (`OracleUSDRaw`, `OracleUSDCallerRaw`, `OracleUSDTransactorRaw`) for low-level access.  
  
**3. Deployment Function:**  
  
The `DeployOracleUSD` function deploys a new instance of the `OracleUSD` contract to the Ethereum blockchain. It takes transaction options (`auth`) and a backend (`backend`) as input and returns the contract address, transaction details, and a bound `OracleUSD` instance.  
  
**4. Method Bindings:**  
  
The code includes bindings for the following contract methods:  
  
*   `GetCurrentPrice()`: Retrieves the current price from the contract.  
*   `Owner()`: Retrieves the owner address of the contract.  
*   `RenounceOwnership()`: Allows the owner to relinquish ownership.  
*   `SetCurrentPrice()`: Allows the owner to set a new price.  
*   `TransferOwnership()`: Allows the owner to transfer ownership to another address.  
  
**5. Event Filtering:**  
  
The code provides functionality for filtering and watching for events emitted by the `OracleUSD` contract:  
  
*   `OwnershipRenounced`: Emitted when the owner renounces ownership.  
*   `OwnershipTransferred`: Emitted when ownership is transferred.  
*   `PriceChanged`: Emitted when the price is updated.  
  
The `Filter...` and `Watch...` functions allow retrieving past events or subscribing to future events, respectively.  Iterators are provided for processing event logs.  
  
<end_of_output>  
```  
  
# blockchain/source/api/Ownable.go  
## Package: api  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (ethereum core functionality)  
*   `github.com/ethereum/go-ethereum/accounts/abi` (ABI parsing)  
*   `github.com/ethereum/go-ethereum/accounts/abi/bind` (Contract binding)  
*   `github.com/ethereum/go-ethereum/common` (Common types like addresses)  
*   `github.com/ethereum/go-ethereum/core/types` (Blockchain types like transactions)  
*   `github.com/ethereum/go-ethereum/event` (Event filtering)  
  
**External Data/Input Sources:**  
  
*   `OwnableABI`: A JSON string representing the Application Binary Interface (ABI) of the Ownable contract. This is used for interacting with the contract.  
*   `OwnableBin`: A hexadecimal string representing the compiled bytecode of the Ownable contract. This is used for deploying the contract.  
*   Ethereum blockchain backend (via `bind.ContractBackend`) for interacting with deployed contracts.  
*   Transaction options (`bind.TransactOpts`) for signing and sending transactions.  
*   Call options (`bind.CallOpts`) for reading contract state.  
*   Filter options (`bind.FilterOpts`) for filtering events.  
  
**TODOs:**  
  
There are no explicit `TODO` comments in the provided code.  
  
### Code Summary  
  
**1. Contract Binding Generation:**  
  
The code provides auto-generated Go bindings for an Ethereum contract named `Ownable`. This includes structures like `Ownable`, `OwnableCaller`, `OwnableTransactor`, and `OwnableFilterer` to facilitate interaction with the contract. The code is generated and should not be manually edited.  
  
**2. Deployment Function:**  
  
The `DeployOwnable` function deploys a new instance of the `Ownable` contract to the Ethereum blockchain. It takes transaction options (`auth`) and a backend (`backend`) as input and returns the contract address, transaction details, and a bound contract instance.  
  
**3. Contract Interface:**  
  
The code defines interfaces for interacting with the `Ownable` contract:  
  
*   `OwnableCaller`: Read-only access to contract functions (e.g., `Owner`).  
*   `OwnableTransactor`: Write access to contract functions (e.g., `RenounceOwnership`, `TransferOwnership`).  
*   `OwnableFilterer`: Filtering and listening for contract events (`OwnershipRenounced`, `OwnershipTransferred`).  
  
**4. Event Handling:**  
  
The code includes structures and functions for filtering and watching for `OwnershipRenounced` and `OwnershipTransferred` events emitted by the `Ownable` contract. Iterators (`OwnableOwnershipRenouncedIterator`, `OwnableOwnershipTransferredIterator`) are provided to process these events.  
  
**5. Raw Access:**  
  
The `OwnableRaw`, `OwnableCallerRaw`, and `OwnableTransactorRaw` types provide low-level access to the contract's underlying functions.  
  
**6. Helper Functions:**  
  
Functions like `NewOwnable`, `NewOwnableCaller`, `NewOwnableTransactor`, and `NewOwnableFilterer` create instances of the contract bindings with different access levels. The `bindOwnable` function handles the underlying ABI parsing and contract binding.  
  
# blockchain/source/api/Pausable.go  
```  
Package: api  
  
Imports:  
- github.com/ethereum/go-ethereum (ethereum)  
- github.com/ethereum/go-ethereum/accounts/abi  
- github.com/ethereum/go-ethereum/accounts/abi/bind  
- github.com/ethereum/go-ethereum/common  
- github.com/ethereum/go-ethereum/core/types  
- github.com/ethereum/go-ethereum/event  
- strings  
  
External Data/Input Sources:  
- PausableABI: A JSON string representing the Application Binary Interface (ABI) of the Pausable contract.  
- PausableBin: A hexadecimal string representing the compiled bytecode of the Pausable contract.  
  
TODOs:  
- None found in the provided code snippet.  
  
Summary:  
  
The code provides a generated Go binding for interacting with an Ethereum contract named "Pausable". This binding allows developers to deploy, call, and listen for events emitted by the Pausable contract.  
  
Key Components:  
- PausableABI: Defines the contract's interface for interacting with it.  
- PausableBin: The compiled bytecode used for deploying the contract.  
- DeployPausable: A function to deploy a new Pausable contract instance.  
- Pausable struct: Contains caller, transactor, and filterer components for interacting with the contract.  
- PausableCaller, PausableTransactor, PausableFilterer: Structs for read-only, write-only, and event filtering operations, respectively.  
- Event Filters: Structures and functions for filtering and watching for OwnershipRenounced, OwnershipTransferred, Pause, and Unpause events.  
- Raw Access: Raw access methods for low-level contract interaction.  
- Session Types: PausableSession, PausableCallerSession, PausableTransactorSession for pre-configured call and transaction options.  
  
The code is auto-generated and should not be manually edited. It provides a comprehensive set of functions for interacting with the Pausable contract, including deployment, method calls, and event monitoring.  
```  
  
# blockchain/source/api/ProfileRegistry.go  
## Package: api  
  
This file defines the auto-generated bindings for the `ProfileRegistry` smart contract. It provides Go structures and methods to interact with the contract, including deployment, calling functions, and listening for events.  
  
**Imports:**  
  
*   `math/big`: For handling large integer values (e.g., uint256).  
*   `strings`: For string manipulation (used in ABI parsing).  
*   `github.com/ethereum/go-ethereum`: Core Ethereum library for interacting with the blockchain.  
*   `github.com/ethereum/go-ethereum/accounts/abi`: For ABI (Application Binary Interface) parsing.  
*   `github.com/ethereum/go-ethereum/accounts/abi/bind`: For binding to smart contracts.  
*   `github.com/ethereum/go-ethereum/common`: For Ethereum-specific data types (e.g., addresses).  
*   `github.com/ethereum/go-ethereum/core/types`: For blockchain data structures (e.g., transactions, logs).  
*   `github.com/ethereum/go-ethereum/event`: For event filtering and subscription.  
  
**External Data/Input Sources:**  
  
*   `ProfileRegistryABI`: The ABI of the smart contract, defining its functions, events, and data structures. This is a string literal containing the ABI in JSON format.  
*   `ProfileRegistryBin`: The compiled bytecode of the smart contract, used for deployment. This is a string literal containing the bytecode in hexadecimal format.  
  
**TODOs:**  
  
There are no explicit `TODO` comments in the provided code.  
  
**Code Summary:**  
  
*   **Deployment:** The `DeployProfileRegistry` function deploys a new instance of the `ProfileRegistry` contract to the blockchain. It takes transaction options and a backend as input and returns the contract address, transaction details, and a bound contract instance.  
*   **Contract Bindings:** The `ProfileRegistry` struct provides access to the contract's functions through `ProfileRegistryCaller` (read-only), `ProfileRegistryTransactor` (write-only), and `ProfileRegistryFilterer` (event filtering).  
*   **Function Calls:** The code includes numerous functions for calling contract methods, such as `GetAttributeCount`, `GetAttributeValue`, `GetCertificate`, `AddValidator`, `CreateCertificate`, `TransferOwnership`, etc. These functions are generated based on the ABI and allow interaction with the contract's logic.  
*   **Event Filtering:** The code provides functions for filtering and subscribing to contract events, such as `CertificateCreated`, `CertificateUpdated`, `OwnershipRenounced`, `OwnershipTransferred`, `Pause`, `Unpause`, `ValidatorCreated`, and `ValidatorDeleted`. These functions allow applications to react to changes in the contract's state.  
*   **Event Iterators:** The code defines iterator structs (e.g., `ProfileRegistryCertificateCreatedIterator`) for iterating over event logs.  
*   **Session Management:** The code includes session structs (`ProfileRegistrySession`, `ProfileRegistryCallerSession`, `ProfileRegistryTransactorSession`) for managing call and transaction options.  
  
# blockchain/source/api/SNM.go  
## Package: api  
  
This package contains auto-generated Go bindings for an Ethereum contract named SNM. The code is generated and should not be manually edited.  
  
**Imports:**  
  
*   `math/big`: For handling large integer values (e.g., token amounts).  
*   `strings`: For string manipulation, specifically used in parsing the ABI.  
*   `github.com/ethereum/go-ethereum`: Core Ethereum library for interacting with the blockchain.  
*   `github.com/ethereum/go-ethereum/accounts/abi`: For encoding and decoding contract ABI.  
*   `github.com/ethereum/go-ethereum/accounts/abi/bind`: For creating Go bindings to Ethereum contracts.  
*   `github.com/ethereum/go-ethereum/common`: For common Ethereum types like addresses.  
*   `github.com/ethereum/go-ethereum/core/types`: For Ethereum transaction and block types.  
*   `github.com/ethereum/go-ethereum/event`: For filtering and watching contract events.  
  
**External Data/Input Sources:**  
  
*   `SNMABI`: A JSON string representing the contract's Application Binary Interface (ABI). This defines the contract's functions, events, and data structures.  
*   `SNMBin`: A hexadecimal string representing the compiled bytecode of the contract. This is used for deploying new instances of the contract.  
  
**TODOs:**  
  
There are no explicit `TODO` comments in the provided code.  
  
**Code Summary:**  
  
*   **Deployment:** The `DeploySNM` function deploys a new contract instance to the blockchain using the provided ABI and bytecode. It returns the contract address, transaction details, and a bound contract instance.  
*   **Contract Bindings:** The `SNM` struct provides access to the contract's functions through `SNMCaller` (read-only), `SNMTransactor` (write-only), and `SNMFilterer` (event filtering) interfaces.  
*   **Function Calls:** The code defines numerous functions for calling contract methods, including `Allowance`, `BalanceOf`, `Decimals`, `Name`, `Owner`, `Symbol`, `TotalSupply`, `Approve`, `DecreaseApproval`, `IncreaseApproval`, `RenounceOwnership`, `Transfer`, `TransferFrom`, and `TransferOwnership`. These functions handle ABI encoding, transaction signing, and result decoding.  
*   **Event Filtering:** The code includes structures and functions for filtering and watching contract events, such as `Approval`, `OwnershipRenounced`, `OwnershipTransferred`, and `Transfer`. These allow external applications to react to specific contract events.  
*   **Session Management:** The `SNMSession`, `SNMCallerSession`, and `SNMTransactorSession` structs provide a way to pre-configure call and transaction options for multiple operations.  
*   **Raw Access:** The `SNMRaw`, `SNMCallerRaw`, and `SNMTransactorRaw` structs provide low-level access to the contract's underlying functions.  
  
# blockchain/source/api/SNMMasterchain.go  
## SNMMasterchain API Package Summary  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `SNMMasterchainABI`: A JSON string representing the Application Binary Interface (ABI) of the SNMMasterchain contract. This is used for interacting with the contract.  
*   `SNMMasterchainBin`: A hexadecimal string representing the compiled bytecode of the SNMMasterchain contract. This is used for deploying new instances of the contract.  
*   `_ico common.Address`: The ICO address used during contract deployment.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
The code generates Go bindings for interacting with an Ethereum smart contract named `SNMMasterchain`. It includes constants for the ABI (`SNMMasterchainABI`) and bytecode (`SNMMasterchainBin`) of the contract. The `DeploySNMMasterchain` function deploys a new contract instance, taking an ICO address as input.  
  
### Contract Structures  
  
The code defines several structures for interacting with the contract:  
  
*   `SNMMasterchain`: The main binding structure, providing access to both read-only and write-only methods.  
*   `SNMMasterchainCaller`: Read-only access to contract functions.  
*   `SNMMasterchainTransactor`: Write access to contract functions.  
*   `SNMMasterchainFilterer`: For filtering and listening to contract events.  
*   Session types (`SNMMasterchainSession`, `SNMMasterchainCallerSession`, `SNMMasterchainTransactorSession`) for pre-configured call/transact options.  
*   Raw access types (`SNMMasterchainRaw`, `SNMMasterchainCallerRaw`, `SNMMasterchainTransactorRaw`) for low-level interaction.  
  
### Function Bindings  
  
The code includes generated functions for calling contract methods:  
  
*   `Allowance`, `BalanceOf`, `Decimals`, `Ico`, `Name`, `Symbol`, `TokensAreFrozen`, `TotalSupply`: Read-only functions.  
*   `Approve`, `Defrost`, `Mint`, `Transfer`, `TransferFrom`: Write functions.  
  
### Event Handling  
  
The code provides structures and functions for filtering and watching for contract events:  
  
*   `SNMMasterchainApprovalIterator`, `SNMMasterchainTransferIterator`: Iterators for events.  
*   `FilterApproval`, `WatchApproval`, `FilterTransfer`, `WatchTransfer`: Functions for filtering and watching events.  
*   `SNMMasterchainApproval`, `SNMMasterchainTransfer`: Event structures.  
  
The code is auto-generated and should not be manually edited. It provides a complete set of bindings for interacting with the SNMMasterchain contract.  
  
# blockchain/source/api/SafeMath.go  
## Package: api  
  
**Imports:**  
  
*   `github.com/ethereum/go-ethereum/accounts/abi`  
*   `github.com/ethereum/go-ethereum/accounts/abi/bind`  
*   `github.com/ethereum/go-ethereum/common`  
*   `github.com/ethereum/go-ethereum/core/types`  
*   `strings`  
  
**External Data/Input Sources:**  
  
*   `SafeMathABI`: A JSON ABI string representing the contract interface.  
*   `SafeMathBin`: The compiled bytecode of the contract, used for deployment.  
*   `common.Address`: Ethereum addresses used for contract deployment and interaction.  
*   `bind.TransactOpts`: Transaction options for deploying and interacting with the contract.  
*   `bind.ContractBackend`: Backend for interacting with the Ethereum blockchain.  
  
**TODOs:**  
  
*   No TODO comments found in the provided code.  
  
**Summary of Code Parts:**  
  
*   **Contract Binding Generation:** This code is auto-generated and provides Go bindings for interacting with an Ethereum contract named `SafeMath`. It includes structures for calling, transacting, and filtering events. The code uses the `go-ethereum` library to handle ABI parsing, contract deployment, and blockchain interaction.  
*   **Deployment Function:** The `DeploySafeMath` function deploys a new `SafeMath` contract to the Ethereum blockchain. It takes transaction options and a backend as input, compiles the contract from the `SafeMathBin` bytecode, and returns the contract address, transaction details, and a bound `SafeMath` instance.  
*   **Contract Structures:** The code defines several structures (`SafeMath`, `SafeMathCaller`, `SafeMathTransactor`, `SafeMathFilterer`, `SafeMathSession`, etc.) that provide different levels of access to the contract's functionality. These structures encapsulate the underlying `bind.BoundContract` and provide methods for calling, transacting, and filtering events.  
*   **Raw Access Structures:** `SafeMathRaw`, `SafeMathCallerRaw`, and `SafeMathTransactorRaw` provide low-level access to the contract's methods, allowing direct interaction with the underlying `bind.BoundContract`.  
*   **Binding Functions:** Functions like `NewSafeMath`, `NewSafeMathCaller`, `NewSafeMathTransactor`, and `NewSafeMathFilterer` create instances of the contract bindings, allowing developers to interact with the deployed contract in different ways.  
*   **Helper Function:** The `bindSafeMath` function is a helper function that binds a generic wrapper to an already deployed contract.  
  
# blockchain/source/api/SimpleGatekeeperWithLimit.go  
```  
Package: api  
  
Imports:  
- math/big  
- strings  
- github.com/ethereum/go-ethereum (multiple sub-packages used: core/types, accounts/abi, accounts/abi/bind, event)  
  
External Data/Input Sources:  
- Ethereum contract ABI (SimpleGatekeeperWithLimitABI) - a JSON string defining the contract's interface.  
- Ethereum contract bytecode (SimpleGatekeeperWithLimitBin) - a hex string representing the compiled contract code.  
- Contract address (used for deployment and interaction).  
- Transaction options (bind.TransactOpts) - used for signing and sending transactions.  
- Backend (bind.ContractBackend) - provides access to the Ethereum blockchain.  
- Input parameters for contract functions (e.g., _token, _freezingTime, _keeper, _limit, _value, _txNumber, _to).  
  
TODOs:  
- None found in the provided code snippet.  
  
Summary of Code Parts:  
  
1.  **Contract Binding Generation:** The code generates Go bindings for an Ethereum smart contract named `SimpleGatekeeperWithLimit`. It uses the provided ABI and bytecode to create instances of the contract that can be deployed and interacted with.  
  
2.  **Data Structures:** Defines several structs (`SimpleGatekeeperWithLimit`, `SimpleGatekeeperWithLimitCaller`, `SimpleGatekeeperWithLimitTransactor`, `SimpleGatekeeperWithLimitFilterer`, and their session/raw counterparts) to represent different ways to interact with the contract (read-only, write-only, event filtering).  
  
3.  **Deployment Function:** The `DeploySimpleGatekeeperWithLimit` function deploys the contract to the blockchain, taking the token address and freezing time as input.  
  
4.  **Function Calls:** The code includes numerous functions for calling contract methods (e.g., `GetCommission`, `GetFreezingTime`, `ChangeKeeperLimit`, `Payin`, `Payout`). These functions are generated based on the ABI and allow interaction with the contract's logic.  
  
5.  **Event Filtering:** The code provides mechanisms for filtering and subscribing to contract events (e.g., `CommissionChanged`, `CommitTx`, `KeeperFreezed`, `OwnershipTransferred`). This allows external applications to react to changes in the contract's state.  
  
6.  **Event Iterators:** Structs like `SimpleGatekeeperWithLimitCommissionChangedIterator` are defined to iterate over emitted events.  
  
The code is a machine-generated binding for an Ethereum smart contract, providing a Go interface for interacting with its functions and events.  
  
<end_of_output>  
```  
  
# blockchain/source/api/SimpleGatekeeperWithLimitLive.go  
## Package: api  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Inputs:**  
  
*   `SimpleGatekeeperWithLimitLiveABI`: JSON ABI definition for the contract.  
*   `SimpleGatekeeperWithLimitLiveBin`: Compiled bytecode for deploying the contract.  
*   `_token`: `common.Address` - Token address passed during contract deployment.  
*   `_freezingTime`: `*big.Int` - Freezing time passed during contract deployment.  
*   Function arguments for various methods (e.g., `_keeper`, `_limit`, `_value`, `_to`, `_txNumber`, `_newOwner`, `_freezingTime`).  
*   Event filters accept slices of `common.Address` and `*big.Int` for filtering events.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
*   **Contract Binding Generation:** The code generates Go bindings for an Ethereum smart contract named `SimpleGatekeeperWithLimitLive`. It uses the provided ABI (`SimpleGatekeeperWithLimitLiveABI`) and bytecode (`SimpleGatekeeperWithLimitLiveBin`) to create deployable and interactable contract wrappers.  
*   **Deployment Function:** `DeploySimpleGatekeeperWithLimitLive` deploys the contract with specified token address and freezing time.  
*   **Contract Structures:** Defines several structures (`SimpleGatekeeperWithLimitLive`, `SimpleGatekeeperWithLimitLiveCaller`, `SimpleGatekeeperWithLimitLiveTransactor`, `SimpleGatekeeperWithLimitLiveFilterer`) to provide different interaction modes (read-only, write-only, event filtering).  
*   **Method Bindings:** Includes numerous functions (e.g., `GetFreezingTime`, `ChangeKeeperLimit`, `Payin`, `Payout`) that map to the contract's methods, allowing interaction with the deployed contract.  
*   **Event Filtering:** Provides functions (`FilterCommitTx`, `WatchCommitTx`, etc.) to filter and subscribe to contract events.  
*   **Event Structures:** Defines structures for each event (e.g., `SimpleGatekeeperWithLimitLiveCommitTx`, `SimpleGatekeeperWithLimitLivePayoutTx`) to hold unpacked event data.  
*   **Raw Access:** Offers raw access to the contract through `SimpleGatekeeperWithLimitLiveRaw` structures for low-level interactions.  
  
# blockchain/source/api/StandardToken.go  
## API Package - StandardToken Binding Summary  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `StandardTokenABI`: A JSON string representing the Application Binary Interface (ABI) of the StandardToken contract. This is used for interacting with the contract.  
*   `StandardTokenBin`: A hexadecimal string representing the compiled bytecode of the StandardToken contract. This is used for deploying new instances of the contract.  
*   Ethereum blockchain backend (via `bind.ContractBackend`) for interacting with deployed contracts.  
*   Transaction options (`bind.TransactOpts`) for signing and sending transactions.  
*   Call options (`bind.CallOpts`) for reading contract state.  
*   Filter options (`bind.FilterOpts`) for filtering events.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
This section defines the ABI and bytecode for a StandardToken contract. The `StandardTokenABI` constant holds the ABI in JSON format, while `StandardTokenBin` contains the compiled bytecode. The `DeployStandardToken` function deploys a new contract instance using the provided ABI and bytecode.  
  
### Core Contract Structures  
  
The code defines several structures for interacting with the StandardToken contract:  
  
*   `StandardToken`: The main binding structure, combining caller, transactor, and filterer interfaces.  
*   `StandardTokenCaller`: Provides read-only access to contract functions.  
*   `StandardTokenTransactor`: Enables sending transactions to modify contract state.  
*   `StandardTokenFilterer`: Allows filtering and listening for contract events.  
*   Session types (`StandardTokenSession`, `StandardTokenCallerSession`, `StandardTokenTransactorSession`) provide pre-configured call and transaction options.  
*   Raw types (`StandardTokenRaw`, `StandardTokenCallerRaw`, `StandardTokenTransactorRaw`) expose low-level access to the contract.  
  
### Function Bindings  
  
The code includes generated functions for calling and transacting with the StandardToken contract:  
  
*   `Allowance`: Retrieves the allowance granted to a spender.  
*   `BalanceOf`: Retrieves the token balance of an address.  
*   `TotalSupply`: Retrieves the total token supply.  
*   `Approve`: Approves a spender to transfer tokens on behalf of the caller.  
*   `DecreaseApproval`: Decreases the approval amount for a spender.  
*   `IncreaseApproval`: Increases the approval amount for a spender.  
*   `Transfer`: Transfers tokens to another address.  
*   `TransferFrom`: Transfers tokens from one address to another on behalf of the owner.  
  
### Event Filtering  
  
The code provides mechanisms for filtering and watching for contract events:  
  
*   `StandardTokenApprovalIterator`: Iterates over `Approval` events.  
*   `StandardTokenTransferIterator`: Iterates over `Transfer` events.  
*   `FilterApproval`: Retrieves past `Approval` events.  
*   `WatchApproval`: Subscribes to new `Approval` events.  
*   `FilterTransfer`: Retrieves past `Transfer` events.  
*   `WatchTransfer`: Subscribes to new `Transfer` events.  
  
# blockchain/source/api/TestnetFaucet.go  
## Package: api  
  
**Package Name:** `api`  
  
**Imports:**  
  
*   `math/big`  
*   `strings`  
*   `github.com/ethereum/go-ethereum` (including `accounts/abi`, `accounts/abi/bind`, `common`, `core/types`, `event`)  
  
**External Data/Input Sources:**  
  
*   `TestnetFaucetABI`: A JSON string representing the Application Binary Interface (ABI) of the `TestnetFaucet` contract. This is used for interacting with the contract.  
*   `TestnetFaucetBin`: A hexadecimal string representing the compiled bytecode of the `TestnetFaucet` contract. This is used for deploying the contract.  
*   `common.Address`: Ethereum addresses used for contract deployment, function calls, and event filtering.  
*   `big.Int`: Used for representing large integer values, such as `mintedAmount` in the `mintToken` function.  
*   `types.Transaction`: Represents Ethereum transactions.  
*   `bind.TransactOpts`: Transaction options for signing and sending transactions.  
*   `bind.CallOpts`: Call options for reading contract state.  
*   `ethereum.Subscription`: Used for watching contract events.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Contract Binding Generation  
  
This code generates Go bindings for an Ethereum smart contract named `TestnetFaucet`. It uses the `github.com/ethereum/go-ethereum` library to create types and functions that allow interaction with the contract. The bindings are generated from the ABI (`TestnetFaucetABI`) and bytecode (`TestnetFaucetBin`) of the contract.  
  
### Core Types  
  
*   `TestnetFaucet`: The main struct representing the contract binding. It includes caller, transactor, and filterer components.  
*   `TestnetFaucetCaller`: Provides read-only access to contract functions.  
*   `TestnetFaucetTransactor`: Provides write access to contract functions.  
*   `TestnetFaucetFilterer`: Allows filtering and listening for contract events.  
*   `TestnetFaucetSession`, `TestnetFaucetCallerSession`, `TestnetFaucetTransactorSession`: Provide pre-configured call and transaction options for easier use.  
*   `TestnetFaucetRaw`, `TestnetFaucetCallerRaw`, `TestnetFaucetTransactorRaw`: Provide low-level access to the contract.  
  
### Deployment Function  
  
*   `DeployTestnetFaucet`: Deploys a new instance of the `TestnetFaucet` contract to the Ethereum blockchain. It takes transaction options and a backend as input and returns the contract address, transaction details, and a bound contract instance.  
  
### Function Bindings  
  
The code includes generated functions for calling contract methods:  
  
*   `GetTokenAddress()`: Returns the address of the token contract.  
*   `Owner()`: Returns the address of the contract owner.  
*   `GetTokens()`: Executes a state-changing function.  
*   `MintToken()`: Mints tokens to a specified address.  
*   `RenounceOwnership()`: Renounces ownership of the contract.  
*   `TransferOwnership()`: Transfers ownership of the contract to a new address.  
  
### Event Filtering  
  
The code provides functions for filtering and watching for contract events:  
  
*   `FilterOwnershipRenounced()`: Filters for `OwnershipRenounced` events.  
*   `WatchOwnershipRenounced()`: Subscribes to `OwnershipRenounced` events.  
*   `FilterOwnershipTransferred()`: Filters for `OwnershipTransferred` events.  
*   `WatchOwnershipTransferred()`: Subscribes to `OwnershipTransferred` events.  
  
The event structures (`TestnetFaucetOwnershipRenounced`, `TestnetFaucetOwnershipTransferred`) contain the event data and the raw log.  
  
