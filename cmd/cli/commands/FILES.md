# cmd/cli/commands/blacklist.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O.  
*   `github.com/ethereum/go-ethereum/crypto`: For cryptographic operations, specifically converting public keys to addresses.  
*   `github.com/sonm-io/core/proto`: For SONM protocol definitions (e.g., `sonm.EthAddress`).  
*   `github.com/sonm-io/core/util`: For utility functions, such as hex-to-address conversion.  
*   `github.com/spf13/cobra`: For building command-line interfaces.  
  
**External Data/Input Sources:**  
  
*   Command-line arguments (addresses for `list` and `remove` commands).  
*   KeyStore (loaded via `loadKeyStoreWrapper` - not shown in this snippet, but referenced).  
*   Blacklist client connection (established via `newBlacklistClient`).  
*   Context with timeout (created by `newTimeoutContext`).  
  
**TODOs:**  
  
*   None found in this snippet.  
  
**Code Summary:**  
  
### Blacklist Management Commands  
  
This code defines a set of Cobra commands for managing a blacklist of addresses. The root command is `blacklist`, with subcommands for listing, removing, and purging entries.  
  
*   **`blacklist list [addr]`**: Retrieves and prints the blacklist for a given owner address. If no address is provided, it defaults to the address derived from the loaded key.  
*   **`blacklist remove <addr>`**: Removes a specified address from the blacklist. Requires one argument (the address to remove).  
*   **`blacklist purge`**: Removes all addresses from the blacklist.  
  
All commands use a blacklist client (`newBlacklistClient`) to interact with the blacklist service. They handle errors during client connection, address conversion, and blacklist operations. The `loadKeyStoreWrapper` function (not shown) is used to load the key store before executing any blacklist commands. The `newTimeoutContext` function creates a context with a timeout to prevent indefinite blocking. The `printBlacklist` and `showOk` functions are used for outputting results. The `printErrorByID` function is used for printing errors.  
  
# cmd/cli/commands/client.go  
## Package: `commands`  
  
**Imports:**  
  
*   `context`  
*   `fmt`  
*   `github.com/sonm-io/core/insonmnia/auth`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `github.com/sonm-io/core/util/xgrpc`  
*   `google.golang.org/grpc`  
  
**External Data/Input Sources:**  
  
*   `nodeAddress()`: Function to retrieve the node address (likely from global flags or configuration).  
*   `insecureFlag`: Boolean flag indicating whether to use insecure gRPC connections.  
*   `creds`: Credentials for gRPC connections (likely from global flags or configuration).  
*   `fullEndpoint`: Parsed node address from `auth.ParseAddr(nodeAddress())`.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
**Summary of Code Parts:**  
  
### gRPC Connection Management (`newClientConn`)  
  
The `newClientConn` function establishes a gRPC connection to a Sonm node. It parses the node address, handles insecure connections based on the `insecureFlag`, and uses `xgrpc.NewClient` to create the connection.  The function relies on globally configured `nodeAddress`, `insecureFlag`, and `creds`.  
  
### Client Creation Functions  
  
The remaining functions (`newWorkerManagementClient`, `newMasterManagementClient`, `newMarketClient`, `newDealsClient`, `newTaskClient`, `newTokenManagementClient`, `newBlacklistClient`, `newProfilesClient`, `newDWHClient`) all follow the same pattern:  
  
1.  Call `newClientConn` to establish a gRPC connection.  
2.  If `newClientConn` fails, return the error.  
3.  Create a client stub for the corresponding Sonm service (WorkerManagement, MasterManagement, Market, etc.) using the established connection.  
4.  Return the client stub.  
  
These functions provide a consistent way to obtain gRPC clients for various Sonm services. They abstract away the connection details and error handling.  
  
# cmd/cli/commands/common.go  
## Package: `commands` Summary  
  
**Package Name:** `commands`  
  
**Imports:**  
  
*   `context`  
*   `crypto/ecdsa`  
*   `encoding/json`  
*   `fmt`  
*   `math/big`  
*   `os`  
*   `time`  
*   `github.com/ethereum/go-ethereum/crypto`  
*   `github.com/mitchellh/go-homedir`  
*   `github.com/sonm-io/core/accounts`  
*   `github.com/sonm-io/core/cmd/cli/config`  
*   `github.com/sonm-io/core/insonmnia/auth`  
*   `github.com/sonm-io/core/util`  
*   `github.com/spf13/cobra`  
*   `google.golang.org/grpc/credentials`  
  
**External Data/Input Sources:**  
  
*   **Configuration File:** Reads configuration from a file specified by the `--config` flag or defaults to a standard location.  
*   **Keystore:** Loads Ethereum keys from a directory specified by the `--keystore` flag or defaults to `~/.sonm/`.  Supports interactive passphrase input if needed.  
*   **Node Address:** Accepts a node endpoint via the `--node` flag, falling back to configuration or `localhost:15030`.  
*   **Command-Line Flags:**  Extensive use of `cobra` flags for controlling output format, timeouts, security settings, and logging.  
*   **Environment Variables:** Potentially used indirectly through the configuration loading process.  
  
**TODOs:**  
  
*   None explicitly found in the provided code snippet.  
  
**Code Summary:**  
  
### Cobra Command Setup  
  
The code defines a `cobra.Command` structure (`rootCmd`) as the entry point for the CLI application. It initializes the command with `SilenceErrors` and `SilenceUsage` set to `true`.  The `version` variable is declared to store the application version.  Numerous flags are defined for controlling various aspects of the CLI's behavior, including node address, output format, timeout, security, keystore location, and configuration file.  Subcommands (worker management, order management, deal management, task management, blacklist management, login, token, version, autocomplete, master, profile) are added to the root command.  
  
### Configuration and Keystore Loading  
  
The `init()` function sets up the `cobra.OnInitialize` hook to load the configuration using `config.NewConfig()`. It also handles the `outputModeJSON` flag, setting `outputModeFlag` accordingly. The `loadKeyStoreWrapper()` function is designed to be used as a `cobra.Command.PreRunE` handler. It initializes the keystore using `accounts.NewMultiKeystore()`, loads the default key (with passphrase prompt if needed), and sets up TLS credentials if the `insecureFlag` is not set.  
  
### Error Handling and Output Formatting  
  
The `commandError` struct is defined to encapsulate errors with a structured JSON format. The `ShowError()` function prints errors in either simple text or JSON format based on the output mode. The `showOk()` function prints "OK" in the same format. The `isSimpleFormat()` function determines whether to use simple text or JSON output.  
  
### Utility Functions  
  
Several utility functions are provided:  
  
*   `nodeAddress()`: Resolves the node address from flags, configuration, or defaults.  
*   `keystorePath()`: Resolves the keystore path from flags, configuration, or defaults.  
*   `initKeystore()`: Initializes the keystore.  
*   `showJSON()`: Prints JSON output.  
*   `newTimeoutContext()`: Creates a context with a timeout.  
*   `argsToBigInts()`: Parses command-line arguments as big integers.  
  
### Key Management  
  
The `getDefaultKey()` function retrieves the default Ethereum key from the keystore, prompting for a passphrase if necessary.  
  
# cmd/cli/commands/completion.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O.  
*   `os`: For operating system functionalities, specifically `os.Stdout`.  
*   `github.com/spf13/cobra`: For command-line application framework.  
  
**External Data/Input Sources:**  
  
*   Command-line arguments: The script takes a single argument specifying the shell type (`bash` or `zsh`).  
*   `rootCmd`: Assumed to be a globally defined `cobra.Command` instance (likely the root command of the application).  
  
**TODOs:**  
  
*   None found in this specific file.  
  
**Code Summary:**  
  
### `autoCompleteCmd` Command Definition  
  
This code defines a `cobra.Command` named `autoCompleteCmd` responsible for generating shell completion scripts for either Bash or Zsh. The command expects one argument: the shell type. It uses a `switch` statement to determine which completion script generation function to call (`rootCmd.GenZshCompletion` or `rootCmd.GenBashCompletion`), writing the output to standard output (`os.Stdout`). If an unknown shell type is provided, it returns an error.  
  
# cmd/cli/commands/deals.go  
## Package: `commands` Summary  
  
**Package Name:** `commands`  
  
**Imports:**  
  
*   `context`  
*   `fmt`  
*   `strings`  
*   `time`  
*   `github.com/sonm-io/core/proto` (as `sonm`)  
*   `github.com/sonm-io/core/util`  
*   `github.com/spf13/cobra`  
*   `golang.org/x/sync/errgroup`  
  
**External Data/Input Sources:**  
  
*   Command-line arguments (deal IDs, ask/bid IDs, durations, prices, blacklist types).  
*   Key store for user address retrieval.  
*   External clients: `dwh`, `market`, `deals` (presumably interacting with a blockchain or distributed system).  
*   Flags defined via `cobra` for configuring command behavior (limit, blacklist type, new duration, new price, force deal, expand deal).  
  
**TODOs:**  
  
No `TODO` comments found in the provided code.  
  
**Code Summaries:**  
  
### Deal Management Commands (`dealRootCmd`)  
  
This section defines the root command for deal-related operations. It includes subcommands for listing, viewing status, opening, quick buying, closing, purging, and managing change requests. The `PersistentPreRunE` function loads the key store before executing any deal commands.  
  
### Deal Listing (`dealListCmd`)  
  
The `dealListCmd` retrieves and prints a list of active deals for the current user. It fetches deals from the `dwh` client, filters by accepted status, and sorts by start time. The number of deals to show is configurable via the `--limit` flag.  
  
### Deal Status (`dealStatusCmd`)  
  
The `dealStatusCmd` retrieves and displays the status of a specific deal. It fetches deal information from the `dealer` client and optionally includes extended order details (ask and bid) if the `--expand` flag is set. It also lists any pending change requests for the deal.  
  
### Deal Opening (`dealOpenCmd`)  
  
The `dealOpenCmd` allows manually opening a deal between two specified orders (ask and bid). It takes the ask and bid IDs as arguments and uses the `deals` client to initiate the deal. The `--force` flag bypasses worker availability checks.  
  
### Quick Deal Buying (`dealQuickBuyCmd`)  
  
The `dealQuickBuyCmd` provides a faster way to open a deal with a given ask ID. It optionally accepts a duration argument. The `--force` flag bypasses worker availability checks. The `--expand` flag includes extended order details.  
  
### Deal Closing (`dealCloseCmd`)  
  
The `dealCloseCmd` closes one or more deals, optionally blacklisting the counterparty (worker or master) based on the `--blacklist` flag. It uses the `dealer` client to finish the deals.  
  
### Deal Purging (`dealPurgeCmd`)  
  
The `dealPurgeCmd` purges all active deals for the current user, optionally blacklisting the counterparty. It uses the `dealer` client to purge the deals.  
  
### Change Request Management (`changeRequestsRoot`)  
  
This section defines subcommands for creating, approving, and canceling change requests for deals.  
  
*   **Create (`changeRequestCreateCmd`):** Creates a change request for a deal, allowing modification of duration and price.  
*   **Approve (`changeRequestApproveCmd`):** Approves a pending change request.  
*   **Cancel (`changeRequestCancelCmd`):** Cancels a pending change request.  
  
# cmd/cli/commands/err_test.go  
## Package: `commands`  
  
**Imports:**  
  
*   `encoding/json`: For JSON marshaling/unmarshaling.  
*   `errors`: For error handling.  
*   `fmt`: For formatted printing.  
*   `testing`: For unit tests.  
*   `github.com/sonm-io/core/cmd/cli/config`: For configuration related to output format.  
*   `github.com/stretchr/testify/assert`: For test assertions.  
  
**External Data/Input Sources:**  
  
*   Configuration (`config.Config`): Used to determine output format (simple or JSON).  
*   Error messages (strings): Passed to `ShowError` function.  
*   Error objects (error interface): Passed to `ShowError` function.  
*   JSON strings: Used as input to `stringToCommandError` for parsing.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
---  
  
### `stringToCommandError` Function  
  
This function attempts to unmarshal a JSON string into a `commandError` struct. It returns the unmarshaled error or an error if unmarshaling fails.  
  
### Test Functions (`TestJsonErrorInternal`, `TestJsonErrorCustom`)  
  
These tests verify that custom and internal errors are correctly serialized into JSON strings using `newCommandError` and `ToJSONString`. They assert that the expected error message and error details are present in the JSON output.  
  
### `myError` Struct  
  
Defines a custom error type with `code` and `msg` fields. Implements the `Error()` method to provide a string representation of the error.  
  
### `ShowError` Function Tests (`TestShowErrorNilErr`, `TestShowErrorWithErr`, `TestShowErrorJsonNilErr`, `TestShowErrorJsonWithErr`)  
  
These tests cover the `ShowError` function's behavior in different output formats (simple and JSON) with and without an underlying error. They assert that the output matches the expected format based on the configuration and error input. The JSON tests also verify that the output can be correctly parsed back into a `commandError` struct using `stringToCommandError`.  
  
# cmd/cli/commands/login.go  
## Package/Component Summary: `commands`  
  
**Package Name:** `commands`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O.  
*   `github.com/ethereum/go-ethereum/crypto`: For cryptographic operations, specifically address generation from public keys.  
*   `github.com/sonm-io/core/accounts`: For keystore management and passphrase handling.  
*   `github.com/sonm-io/core/util`: For utility functions, including hex to address conversion.  
*   `github.com/spf13/cobra`: For command-line interface (CLI) functionality.  
  
**External Data/Input Sources:**  
  
*   **Command-line arguments:** The `loginCmd` accepts an optional address (`addr`) as an argument.  
*   **Password Input:** The command can take a password via the `--password` flag or interactively prompt the user for a passphrase using `accounts.NewInteractivePassPhraser()`.  
*   **Configuration (`cfg`):** The code reads and writes to a global configuration object (`cfg`) to store the Ethereum passphrase and keystore path.  
*   **Keystore:** The code interacts with a keystore (Ethereum key storage) located at a path determined by `keystorePath()`.  
  
**TODOs:**  
  
*   No explicit `TODO` comments are present in the provided code.  
  
---  
  
### Code Sections Summary:  
  
**1. Command Definition (`loginCmd`)**  
  
This section defines the `login` command using `cobra`. The command allows users to either open an existing Ethereum key or generate a new one. It handles both explicit password input via the `--password` flag and interactive passphrase prompts.  
  
**2. Keystore Initialization (`initKeystore`)**  
  
The `initKeystore` function initializes the keystore based on the provided passphrase reader. It retrieves the keystore path using `keystorePath()`.  
  
**3. Address Handling (with/without arguments)**  
  
If an address is provided as an argument, the code attempts to set it as the default key in the keystore. It decrypts the key using the provided passphrase and updates the configuration (`cfg`) with the passphrase and keystore path. If the address is not found, it lists available addresses.  
  
If no address is provided, the code checks if the keystore is empty. If it is, it generates a new key, prompts for a passphrase, and sets the new key as the default. If the keystore is not empty, it displays the default key (if set) or lists all available keys.  
  
**4. Passphrase Management**  
  
The code handles passphrase retrieval using `accounts.PassPhraser` interfaces. It either uses a static passphrase provided via the `--password` flag or prompts the user interactively. The passphrase is stored in the configuration (`cfg`) for future use.  
  
**5. Key Generation and Default Setting**  
  
If a new key is generated, the code uses `ks.GenerateWithPassword()` to create the key and sets it as the default in the keystore. The passphrase is also stored in the configuration.  
  
---  
  
# cmd/cli/commands/master.go  
```markdown  
## Package: commands  
  
**Imports:**  
  
*   `fmt`: For formatted I/O.  
*   `github.com/ethereum/go-ethereum/common`: For Ethereum address handling.  
*   `github.com/ethereum/go-ethereum/crypto`: For cryptographic operations (e.g., public key to address conversion).  
*   `github.com/sonm-io/core/proto`: For SONM protocol definitions (specifically `sonm.EthAddress`).  
*   `github.com/sonm-io/core/util`: For utility functions (e.g., hex to address conversion).  
*   `github.com/spf13/cobra`: For building command-line interfaces.  
  
**External Data/Input Sources:**  
  
*   **Command-line arguments:** The commands accept Ethereum addresses (worker and master) as input via command-line arguments. These are converted from hex strings using `util.HexToAddress`.  
*   **KeyStore:** The commands rely on a key store (loaded via `loadKeyStoreWrapper`) to obtain the default key for the master. This key is used to derive the master's Ethereum address.  
*   **Master Management Client:** The commands interact with a `MasterManagementClient` (created by `newMasterManagementClient`) to perform operations like listing workers, confirming registrations, and removing relationships. This client likely connects to a remote service.  
*   **Context with Timeout:** Each command uses a context with a timeout (`newTimeoutContext`) to prevent indefinite blocking.  
  
**TODOs:**  
  
*   No explicit `TODO` comments found in the provided code.  
  
### Command Structure  
  
The package defines a set of `cobra` commands for managing master and worker relationships within the SONM network. The root command is `master`, and it has the following subcommands:  
  
*   `list`: Lists registered workers associated with a given master address. If no master address is provided, it uses the address derived from the default key.  
*   `confirm`: Confirms a pending worker registration request. Requires a worker Ethereum address as input.  
*   `remove_worker`: Removes a registered worker from the master's list. Requires a worker Ethereum address as input.  
*   `remove_master`: Removes the current master from a specified master's list. Requires a master Ethereum address as input.  
  
### Core Logic  
  
The core logic revolves around interacting with the `MasterManagementClient` to perform the desired operations. The commands first validate input addresses, then call the appropriate method on the client (e.g., `WorkersList`, `WorkerConfirm`, `WorkerRemove`). Error handling is present, with errors being wrapped in informative messages. The `masterRemove` function encapsulates the worker removal logic, used by both `remove_worker` and `remove_master`.  
  
### Key Derivation  
  
The commands derive the master's Ethereum address from the default key obtained from the key store. This key is used to identify the master in interactions with the `MasterManagementClient`.  
  
<end_of_output>  
```  
  
# cmd/cli/commands/orders.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt` (standard library): For formatted I/O.  
*   `github.com/sonm-io/core/cmd/cli/task_config`: For loading task configurations from files.  
*   `github.com/sonm-io/core/proto`: Contains protocol definitions (likely gRPC or similar).  
*   `github.com/spf13/cobra`: For building command-line interfaces.  
  
**External Data/Input Sources:**  
  
*   **Bid Order YAML File:** The `orderCreateCmd` takes a path to a YAML file containing a bid order definition. This file is parsed using `task_config.LoadFromFile`.  
*   **Order IDs (String):** The `orderStatusCmd`, `orderCancelCmd` take order IDs as string arguments.  
*   **Marketplace Client:** All commands interact with a marketplace client (`newMarketClient`), which presumably connects to an external service.  
  
**TODOs:**  
  
*   No TODO comments found in the provided code.  
  
---  
  
### Order Management Root Command (`orderRootCmd`)  
  
This command serves as the root for all order-related subcommands. It uses `cobra` to define the command structure and includes a `PersistentPreRunE` function (`loadKeyStoreWrapper`) which likely handles authentication or key loading before any subcommand is executed.  
  
### Order Listing (`orderListCmd`)  
  
This command retrieves and displays a list of active orders from the marketplace. It takes an optional `--limit` flag to control the number of orders returned. It uses the `GetOrders` method of the marketplace client to fetch the orders.  
  
### Order Status (`orderStatusCmd`)  
  
This command retrieves and displays the details of a specific order by its ID. It takes the order ID as a required argument. It uses the `GetOrderByID` method of the marketplace client to fetch the order details.  
  
### Order Creation (`orderCreateCmd`)  
  
This command creates a new bid order on the marketplace. It takes the path to a YAML file containing the order definition as an argument. It uses `task_config.LoadFromFile` to parse the YAML file and then calls the `CreateOrder` method of the marketplace client to submit the order.  
  
### Order Cancellation (`orderCancelCmd`)  
  
This command cancels one or more orders on the marketplace. It takes one or more order IDs as arguments. It converts the order IDs to `sonm.BigInt` and then calls the `CancelOrders` method of the marketplace client to cancel the orders.  
  
### Order Purging (`orderPurgeCmd`)  
  
This command removes all orders associated with the current user from the marketplace. It calls the `PurgeVerbose` method of the marketplace client to purge the orders.  
  
# cmd/cli/commands/printers.go  
```  
Package: commands  
  
Imports:  
- encoding/json  
- fmt  
- math/big  
- reflect  
- strings  
- time  
- github.com/ethereum/go-ethereum/common  
- github.com/ethereum/go-ethereum/crypto  
- github.com/olekukonko/tablewriter  
- github.com/sonm-io/core/proto (as sonm)  
- github.com/sonm-io/core/util  
- github.com/sonm-io/core/util/datasize  
- github.com/spf13/cobra  
- gopkg.in/yaml.v2  
  
External Data/Input Sources:  
- sonm.TaskStatusReply, sonm.NetworkSpec, sonm.DealInfoReply, sonm.StatusReply, sonm.Benchmark, sonm.DevicesReply, sonm.Order, sonm.Deal, sonm.ErrorByID, sonm.ErrorByStringID, sonm.BalanceReply, sonm.BigInt, sonm.BlacklistReply, sonm.WorkerListReply, sonm.Profile  
- Command-line flags (via cobra)  
- YAML and JSON data for configuration and output  
- Ethereum addresses (common.Address)  
  
TODOs:  
- Breaking issue #1225 (related to typeEraseWithFieldMap function)  
- Will be removed in the next minor version update #1499 (related to printBalanceInfo function)  
  
Code Summary:  
  
Printer Interface: Defines a generic printer interface (Printer) and an indented printer (IndentPrinter) for formatted output.  
  
Data Printing Functions:  
- printTaskStatus: Prints task status details in simple or JSON format. Handles YAML marshaling for tags.  
- printNetworkSpec: Prints network specifications in YAML format.  
- printTaskStatuses: Prints multiple task statuses.  
- printNodeTaskStatus: Prints node task status (running and completed tasks).  
- printWorkerStatus: Prints worker status details.  
- printBenchmarkGroup: Prints benchmark results.  
- printDeviceList: Prints device information (CPU, RAM, GPUs, network, storage).  
- printOrdersList: Prints a list of orders in a table or JSON format.  
- printOrderDetails: Prints detailed order information.  
- printAskList: Prints ask plans in YAML or JSON format.  
- printVersion: Prints the application version.  
- printDealsList: Prints deal list with expenses per hour.  
- printDealInfo: Prints detailed deal information.  
- printErrorByID: Prints errors by ID.  
- printID: Prints an ID.  
- printTaskStart: Prints task start information.  
- printBalanceInfo: Prints balance information.  
- printMarketAllowance: Prints market allowance.  
- printBlacklist: Prints blacklist addresses.  
- printWorkersList: Prints worker list.  
- printProfileInfo: Prints profile information.  
  
Utility Functions:  
- typeEraseWithFieldMap: Converts a struct to a YAML-compatible map, handling specific field transformations.  
- dealsExpensesPerHour: Calculates expenses per hour for deals.  
- dealType: Determines the type of a deal (buy, sell, or both).  
- getDealCounterpartyString: Returns the counterparty address for a deal.  
  
Format Control:  
- isSimpleFormat: Checks if simple format output is enabled.  
- showJSON: Prints data in JSON format.  
  
The package provides a comprehensive set of functions for printing various data structures related to the SONM platform in both human-readable (simple) and JSON formats. It includes formatting, indentation, and error handling.  
<end_of_output>  
```  
  
# cmd/cli/commands/printers_test.go  
## Package: `commands`  
  
**Imports:**  
  
*   `os`  
*   `testing`  
*   `github.com/ethereum/go-ethereum/common`  
*   `github.com/ethereum/go-ethereum/crypto`  
*   `github.com/sonm-io/core/accounts`  
*   `github.com/sonm-io/core/cmd/cli/config`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `github.com/stretchr/testify/assert`  
*   `github.com/stretchr/testify/require`  
  
**External Data/Input Sources:**  
  
*   Temporary directory created using `os.TempDir()` for keystore generation.  
*   Configuration loaded from `config.Config` struct, including output format (`config.OutputModeJSON`, `config.OutputModeSimple`) and Ethereum configuration (`Eth` field with passphrase).  
*   Ethereum addresses and big integers are used, potentially sourced from external contracts or user input.  
*   Test data including Ethereum addresses (`0x111`, `0x222`, `0x928cA7817FE2eBAC8C41e9dEF8EA6c09ffbd385A`) and big integer values.  
  
**TODOs:**  
  
*   No TODO comments found in the provided code.  
  
### Code Summaries:  
  
**1. `TestJsonOutputForOrder`:**  
  
This test verifies that order prices are serialized as strings in JSON output, rather than using the `abs` and `neg` parts of `sonm.BigInt`. It sets the output format to JSON, creates a test order with a large price, prints the order list, and asserts that the output matches the expected JSON string.  
  
**2. `TestDealInfoWithZeroDuration`:**  
  
This test checks the output of deal information when the deal duration is zero. It generates a temporary keystore, creates a deal with zero start and end times, sets the output format to simple, and prints the deal info. It asserts that the output contains "Duration: 0s".  
  
**3. `TestFlags`:**  
  
This test verifies the behavior of printer flags related to warning suppression. It asserts that the default printer flags do not suppress warnings, while `suppressWarnings` flag does.  
  
**4. `TestExpensesPerHour`:**  
  
This test calculates the total expenses per hour for a given Ethereum address based on a list of deals. It creates a list of deals, calculates the total expenses for asks and bids, and asserts that the calculated values match the expected results.  
  
# cmd/cli/commands/profiles.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt`  
*   `github.com/ethereum/go-ethereum/crypto`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `github.com/sonm-io/core/util`  
*   `github.com/spf13/cobra`  
*   `google.golang.org/grpc/codes`  
*   `google.golang.org/grpc/status`  
  
**External Data/Input Sources:**  
  
*   Command-line arguments (address for `status` command, attribute ID for `remove-attr` command).  
*   KeyStore (loaded via `loadKeyStoreWrapper` - not shown in this snippet, but referenced).  
*   gRPC service connection to a `profiles` service (accessed via `newProfilesClient`).  
*   Hex-encoded Ethereum addresses (converted using `util.HexToAddress`).  
  
**TODOs:**  
  
*   No explicit `TODO` comments found in this snippet.  
  
---  
  
### `profileRootCmd`  
  
This is the root command for the `profile` subcommand group. It uses the `cobra` library to define a command structure. The `PersistentPreRunE` function `loadKeyStoreWrapper` (not defined here) is executed before any subcommand is run, likely handling key loading.  
  
### `profileStatusCmd`  
  
This command retrieves and displays profile details for a given Ethereum address. It takes an optional address as an argument; if none is provided, it uses the address derived from the loaded key. It connects to a gRPC `profiles` service, calls the `Status` method, and prints the profile information. Error handling includes checking for `NotFound` errors from the gRPC service.  
  
### `profileRemoveAttrCmd`  
  
This command removes an attribute from the user's profile. It takes an attribute ID as an argument, connects to the gRPC `profiles` service, and calls the `RemoveAttribute` method. It handles potential errors during the gRPC call and displays a success message if the operation is successful. The attribute ID is converted to a `sonm.BigInt` before being sent to the service.  
  
# cmd/cli/commands/tasks.go  
## Package: `commands` Summary  
  
**Package Name:** `commands`  
  
**Imports:**  
  
*   `bufio`  
*   `bytes`  
*   `context`  
*   `fmt`  
*   `io`  
*   `math/big`  
*   `os`  
*   `strconv`  
*   `strings`  
*   `github.com/docker/docker/pkg/stdcopy`  
*   `github.com/ethereum/go-ethereum/crypto`  
*   `github.com/gosuri/uiprogress`  
*   `github.com/sonm-io/core/cmd/cli/task_config`  
*   `github.com/sonm-io/core/insonmnia/structs`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `github.com/sonm-io/core/util`  
*   `github.com/spf13/cobra`  
*   `google.golang.org/grpc/metadata`  
  
**External Data/Input Sources:**  
  
*   Command-line arguments (deal ID, task ID, file paths, log types, timestamps, etc.)  
*   Task configuration files (YAML format) loaded via `task_config.LoadConfig`.  
*   Remote gRPC services (accessed via `newTaskClient` and `newDealsClient`).  
*   File system (for reading task archives during `push` and outputting logs during `pull`).  
*   Standard input/output streams for logging.  
  
**TODOs:**  
  
No TODO comments found in the provided code.  
  
**Code Sections Summary:**  
  
*   **Command Initialization (`init()`):**  Sets up Cobra command flags for task management (logs, list, start, stop, pull, push, join network).  Flags control log output, timestamps, following logs, tailing, and details.  
*   **Context Handling (`newDealContext()`):** Creates a gRPC context with a `deal` metadata entry for routing requests.  
*   **Deal ID Retrieval (`getActiveDealIDs()`):** Fetches active deal IDs from a remote service, filtering to include only deals where the current user is a consumer.  
*   **Task List Command (`taskListCmd`):** Lists active tasks for a given deal ID. Supports listing all deals or a specific deal.  Prints task status in simple or JSON format.  
*   **Task Start Command (`taskStartCmd`):** Starts a task by loading a task definition from a YAML file and sending a `StartTaskRequest` to a remote service.  
*   **Task Status Command (`taskStatusCmd`):** Retrieves the status of a task from a remote service.  
*   **Task Join Network Command (`taskJoinNetworkCmd`):** Retrieves network specifications for joining a task's network from a remote service.  
*   **Log Handling (`logWriter`):**  Wraps an `io.Writer` to prepend a prefix to log lines.  
*   **Log Type Parsing (`parseType()`):** Converts a string log type (stdout, stderr, both) to a `sonm.TaskLogsRequest_Type` enum.  
*   **Task Logs Command (`taskLogsCmd`):** Retrieves task logs from a remote service, supporting filtering by log type, timestamp, and following logs in real-time.  
*   **Task Stop Command (`taskStopCmd`):** Stops a task by sending a `StopTask` request to a remote service.  
*   **Task Purge Command (`taskPurgeCmd`):** Purges all tasks running on a given deal.  
*   **Task Pull Command (`taskPullCmd`):** Pulls a committed image from a completed task, streaming the image data to stdout or a specified file. Includes a progress bar.  
*   **Task Push Command (`taskPushCmd`):** Pushes an image from the filesystem to a remote service, streaming the image data in chunks. Includes a progress bar.  
  
# cmd/cli/commands/tokens.go  
## Package: `commands`  
  
**Imports:**  
  
*   `context`: For managing request contexts, including timeouts.  
*   `fmt`: For formatted I/O.  
*   `math/big`: For arbitrary-precision arithmetic, used for token amounts.  
*   `time`: For time-related operations, such as setting timeouts.  
*   `github.com/sonm-io/core/proto`: Contains SONM-specific protobuf definitions, including `sonm.EthAddress` and other related structures.  
*   `github.com/spf13/cobra`: For building command-line interfaces.  
*   `github.com/tcnksm/go-input`: For interactive user input (e.g., confirmation prompts).  
  
**External Data/Input Sources:**  
  
*   **Command-line arguments:** The commands accept arguments such as addresses, amounts, and flags (e.g., `--force`).  
*   **Keystore:** The code retrieves the default Ethereum address from the keystore for operations where the sender is not explicitly specified.  
*   **SONM blockchain/masterchain:** The commands interact with the SONM blockchain (sidechain) and potentially the masterchain through a `TokenManagementClient`.  
*   **User input:** The `tokenTransferCmd` prompts the user for confirmation before transferring tokens unless the `--force` flag is used.  
  
**TODOs:**  
  
*   No explicit `TODO` comments are present in the provided code.  
  
### Command Structure  
  
The package defines a set of Cobra commands for managing SONM tokens. The root command is `tokenRootCmd`, which serves as a parent for the following subcommands:  
  
*   `tokenBalanceCmd`: Displays the SONM token balance for a given address (or the default address if none is provided).  
*   `tokenDepositCmd`: Transfers SNM tokens from the masterchain to the SONM blockchain.  
*   `tokenWithdrawCmd`: Transfers SNM tokens from the SONM blockchain to the masterchain.  
*   `tokenMarketAllowanceCmd`: Shows the current allowance for marketplace operations on the sidechain.  
*   `tokenTransferCmd`: Transfers SNM tokens between accounts on the sidechain. This command includes a confirmation prompt unless the `--force` flag is used.  
  
### Key Functions  
  
*   `newTokenManagementClient(ctx)`: Creates a connection to the token management service.  
*   `parseSNMValueInput(arg)`: Parses a string argument representing an SNM token amount, converting it to a `big.Int`.  
*   `showTransferPrompt(amount, to)`: Prompts the user for confirmation before transferring tokens.  
*   `newTimeoutContext()`: Creates a context with a default timeout.  
  
### Error Handling  
  
The commands include error handling for various operations, such as parsing addresses, creating client connections, and executing token transfers. Errors are wrapped with informative messages before being returned.  
  
# cmd/cli/commands/version.go  
## Package: `commands`  
  
**Imports:**  
  
*   `github.com/spf13/cobra`  
  
**External Data/Input Sources:**  
  
*   The `version` variable (presumably defined elsewhere) is used by the `printVersion` function. The source of this variable is not visible in this snippet.  
  
**TODOs:**  
  
*   None found in this snippet.  
  
**Code Summary:**  
  
### `versionCmd` Command Definition  
  
This code defines a Cobra command named `version`. The command's `Use` is "version", and its `Short` description is "Show version". When executed, the `RunE` function calls `printVersion` (presumably a function defined elsewhere) passing the command itself and the `version` variable as arguments. The function returns `nil`, indicating successful execution.  
  
# cmd/cli/commands/version_test.go  
## Package: `commands`  
  
**Imports:**  
  
*   `bytes`: For working with byte buffers.  
*   `encoding/json`: For JSON encoding/decoding.  
*   `testing`: For unit testing.  
*   `github.com/sonm-io/core/cmd/cli/config`: For configuration related to output format.  
*   `github.com/stretchr/testify/assert`: For assertions in tests.  
  
**External Data/Input Sources:**  
  
*   `version` variable (presumably defined elsewhere): Used to represent the application version.  
*   `rootCmd` variable (presumably defined elsewhere): Represents the root command for the CLI.  
*   `cfg` variable (presumably defined elsewhere): Represents the configuration object.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
**Summary of Code Parts:**  
  
*   **`initRootCmd` Function:** This function initializes the root command (`rootCmd`) for testing purposes. It resets commands and flags, sets arguments to an empty string, and sets the output to a byte buffer. It returns the buffer for capturing command output.  
*   **`TestGetVersionCmdSimple` Function:** This test case verifies that the version command outputs the version string in a simple text format when `cfg.OutFormat` is set to `config.OutputModeSimple`. It calls `printVersion` and asserts that the output contains the expected version string.  
*   **`TestGetVersionCmdJson` Function:** This test case verifies that the version command outputs the version in JSON format when `cfg.OutFormat` is set to `config.OutputModeJSON`. It calls `printVersion`, unmarshals the output into a map, and asserts that the map contains "version" and "platform" keys.  
  
This file contains unit tests for the version command within the `commands` package. The tests cover both simple text and JSON output formats. The tests rely on external configuration (`cfg`) and command (`rootCmd`) variables.  
  
# cmd/cli/commands/worker.go  
## Package: `commands` Summary  
  
**Package Name:** `commands`  
  
**Imports:**  
  
*   `context`  
*   `encoding/json`  
*   `errors`  
*   `fmt`  
*   `io`  
*   `strconv`  
*   `time`  
*   `github.com/ethereum/go-ethereum/crypto`  
*   `github.com/sonm-io/core/insonmnia/auth`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `github.com/sonm-io/core/util`  
*   `github.com/spf13/cobra`  
*   `google.golang.org/grpc`  
*   `google.golang.org/grpc/metadata`  
*   `gopkg.in/yaml.v2`  
  
**External Data/Input Sources:**  
  
*   Configuration (`cfg`) - likely loaded from a file or environment variables. Contains `WorkerAddr`.  
*   Command-line flags, particularly `worker-address`.  
*   gRPC connection to a worker management service.  
*   Key store (loaded via `loadKeyStoreWrapper`).  
*   User-provided addresses (ETH addresses) for commands like `switch`, `add-capability`, and `remove-capability`.  
*   Time inputs (either as timestamps or durations) for `maintenance`.  
  
**TODOs:**  
  
*   `// todo: proper printer` in `workerNextMaintenanceCmd` - indicates a need for a more structured output format for the next maintenance time.  
  
**Code Summaries:**  
  
### Worker Management Setup (`workerPreRunE`)  
  
This function initializes the gRPC connection to the worker management service. It loads the key store, sets the worker address (either from a flag, config, or derived from the default key), and adds metadata to the context for authentication. It handles potential errors during connection setup.  
  
### Worker Lifecycle (`workerPostRun`)  
  
This function cleans up the gRPC connection by canceling the context when the command finishes.  
  
### Command Initialization (`init`)  
  
This function registers subcommands under the `worker` command using `cobra`. These subcommands include status, switching workers, scheduling maintenance, debugging, and managing capabilities.  
  
### Worker Status (`workerStatusCmd`)  
  
Retrieves and prints the worker's status from the gRPC service.  
  
### Worker Switching (`workerSwitchCmd`)  
  
Allows changing the current worker address by updating the configuration.  
  
### Scheduling Maintenance (`workerScheduleMaintenanceCmd`)  
  
Schedules worker maintenance at a specified time (either absolute or relative).  
  
### Next Maintenance (`workerNextMaintenanceCmd`)  
  
Retrieves and prints the next scheduled maintenance time. The output format is either simple or JSON.  
  
### Current Worker (`workerCurrentCmd`)  
  
Displays the current worker's address, handling cases where it's not set or invalid.  
  
### Debug State (`workerDebugStateCmd`)  
  
Retrieves and prints the worker's debug state in YAML or JSON format.  
  
### Logs (`workerLogs`)  
  
Subscribes to and prints the worker's logs in real-time.  
  
### Capability Management (`workerAddCapabilityCmd`, `workerRemoveCapabilityCmd`)  
  
Adds or removes worker-specific capabilities for other users.  
  
### Worker Config (`workerConfigCmd`)  
  
Fetches and prints the worker's configuration in YAML or JSON format.  
  
# cmd/cli/commands/worker_askplans.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt` (standard library)  
*   `github.com/sonm-io/core/cmd/cli/task_config`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `github.com/spf13/cobra`  
  
**External Data/Input Sources:**  
  
*   `worker` (presumably an external worker component, used for interacting with ask plans)  
*   `workerCtx` (context for the worker component)  
*   YAML file path for creating ask plans (`ask_plan.yaml`)  
*   Ask order ID for removing plans (`order_id`)  
  
**TODOs:**  
  
*   No TODO comments found in the provided code.  
  
---  
  
### `askPlansRootCmd`  
  
This command serves as the root for all ask plan operations. It doesn't perform any action itself but groups subcommands for listing, creating, removing, and purging ask plans.  
  
### `askPlanListCmd`  
  
This command retrieves and prints a list of existing ask plans from the worker using `worker.AskPlans`. It handles potential errors during the retrieval process.  
  
### `askPlanCreateCmd`  
  
This command creates a new ask plan by loading a definition from a YAML file using `task_config.LoadFromFile`. It then sends the plan to the worker using `worker.CreateAskPlan` and prints the newly created plan's ID.  
  
### `askPlanRemoveCmd`  
  
This command removes an ask plan by its ID using `worker.RemoveAskPlan`. It handles potential errors during the removal process.  
  
### `askPlanPurgeCmd`  
  
This command purges all existing ask plans on the worker using `worker.PurgeAskPlansDetailed`. It prints any errors encountered during the purge operation.  
  
# cmd/cli/commands/worker_benchmarks.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O.  
*   `strconv`: For string conversion.  
*   `github.com/sonm-io/core/proto`: Contains protobuf definitions, specifically `sonm.Empty` and `sonm.NumericID`.  
*   `github.com/spf13/cobra`: For building command-line applications.  
  
**External Data/Input Sources:**  
  
*   `workerCtx`: Context for worker operations (presumably defined elsewhere in the package or a parent package).  
*   `worker`: An object with methods `PurgeBenchmarks` and `RemoveBenchmark` (presumably defined elsewhere).  
*   Command-line arguments: Specifically, the benchmark ID for the `remove` command.  
  
**TODOs:**  
  
*   None found in this file.  
  
---  
  
### `benchmarkRootCmd`  
  
This is the root command for benchmark-related operations. It serves as a container for subcommands like `purge` and `remove`.  
  
### `workerPurgeBenchmarksCmd`  
  
This command purges all benchmarks from the worker's cache. It calls the `worker.PurgeBenchmarks` method with the `workerCtx` and an empty protobuf message.  Error handling is present, and a success message is displayed if the operation completes without errors.  
  
### `workerRemoveBenchmarksCmd`  
  
This command removes a specific benchmark from the cache, identified by its ID. It parses the provided ID from the command-line arguments using `strconv.ParseUint`. It then calls the `worker.RemoveBenchmark` method with the `workerCtx` and a `sonm.NumericID` containing the parsed ID. Error handling is included for both ID parsing and benchmark removal, and a success message is displayed on completion.  
  
# cmd/cli/commands/worker_devices.go  
## Package/Component Name: `commands`  
  
**Imports:**  
  
*   `fmt` (standard library for formatted I/O)  
*   `github.com/sonm-io/core/proto` (SONM core protocol definitions)  
*   `github.com/spf13/cobra` (Cobra CLI framework)  
  
**External Data/Input Sources:**  
  
*   `workerCtx`: Context likely passed from elsewhere in the application, presumably containing worker-related configuration or state.  
*   `sonm.Empty{}`: An empty protocol buffer message, likely used as a placeholder or default input for the RPC calls.  
*   `worker.Devices()` and `worker.FreeDevices()`: Functions from a `worker` module (not defined in this snippet) that retrieve device lists.  
  
**TODOs:**  
  
*   None found in this snippet.  
  
---  
  
### `workerDevicesCmd`  
  
This Cobra command, `devices`, retrieves and prints a list of all worker devices. It calls `worker.Devices()` with the `workerCtx` and an empty protocol buffer message. If the call fails, it returns an error message. The `printDeviceList()` function (not defined here) is then called to display the device list.  
  
### `workerFreeDevicesCmd`  
  
This Cobra command, `free_devices`, retrieves and prints a list of worker devices with remaining resources. It calls `worker.FreeDevices()` with the `workerCtx` and an empty protocol buffer message. Similar to `workerDevicesCmd`, it handles errors and uses `printDeviceList()` to display the results.  
  
# cmd/cli/commands/worker_metrics.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt` (standard library): For formatted I/O.  
*   `github.com/sonm-io/core/proto`: Contains the `sonm` package, likely defining protocol buffer structures.  
*   `github.com/spf13/cobra`: For building command-line applications.  
*   `gopkg.in/yaml.v2`: For YAML serialization.  
  
**External Data/Input Sources:**  
  
*   `workerCtx`: Context variable, presumably passed from elsewhere in the application.  
*   `sonm.WorkerMetricsRequest{}`: An empty request object for fetching worker metrics.  
*   `isSimpleFormat()`: A function (not defined in this snippet) that determines whether to output in YAML or JSON.  
  
**TODOs:**  
  
*   None found in this snippet.  
  
**Code Summary:**  
  
### Command Definition: `workerMetricsCmd`  
  
This code defines a `cobra.Command` named `metrics` under the `commands` package. The command's purpose is to retrieve and display worker hardware monitoring metrics.  
  
### Execution Logic (`RunE` function)  
  
The `RunE` function fetches worker metrics using `worker.Metrics(workerCtx, &sonm.WorkerMetricsRequest{})`. If an error occurs during metric retrieval, it's wrapped and returned.  
  
### Output Formatting  
  
The output format is determined by the `isSimpleFormat()` function. If `true`, the metrics are serialized into YAML using `yaml.Marshal()` and printed to stdout. Otherwise, the `showJSON()` function (not defined in this snippet) is called to output the metrics in JSON format.  
  
# cmd/cli/commands/worker_tasks.go  
## Package: `commands`  
  
**Imports:**  
  
*   `fmt` (standard library)  
*   `github.com/sonm-io/core/proto` (external dependency - SONM core proto definitions)  
*   `github.com/spf13/cobra` (external dependency - Cobra CLI framework)  
  
**External Data/Input Sources:**  
  
*   `workerCtx`: Context related to the worker (presumably passed from elsewhere in the application).  
*   `sonm.Empty{}`: An empty proto message used as input to the `worker.Tasks` function.  
*   `worker.Tasks()`: Function call to get the task list from the worker.  
  
**TODOs:**  
  
*   None found in this snippet.  
  
**Code Summary:**  
  
### `workerTasksCmd` Command Definition  
  
This code defines a Cobra command named `tasks` intended to display the tasks currently running on a Worker node. The `RunE` function executes when the command is invoked. It calls `worker.Tasks()` to retrieve the task list, handles potential errors during retrieval, and then prints the task statuses using the `printTaskStatuses()` function (not defined in this snippet). The command takes no arguments.  
  
