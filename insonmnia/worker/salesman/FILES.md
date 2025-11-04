# insonmnia/worker/salesman/options.go  
## Salesman Package Component Summary  
  
**Package Name:** `salesman`  
  
**Imports:**  
  
*   `context`  
*   `crypto/ecdsa`  
*   `errors`  
*   `github.com/sonm-io/core/blockchain`  
*   `github.com/sonm-io/core/insonmnia/cgroups`  
*   `github.com/sonm-io/core/insonmnia/hardware`  
*   `github.com/sonm-io/core/insonmnia/matcher`  
*   `github.com/sonm-io/core/insonmnia/resource`  
*   `github.com/sonm-io/core/insonmnia/state`  
*   `github.com/sonm-io/core/insonmnia/worker/network`  
*   `github.com/sonm-io/core/proto`  
*   `github.com/sonm-io/core/util/multierror`  
*   `go.uber.org/zap`  
  
**External Data/Input Sources:**  
  
*   `blockchain.API`: Ethereum blockchain interface.  
*   `state.Storage`: Storage for application state.  
*   `resource.Scheduler`: Resource scheduling component.  
*   `hardware.Hardware`: Hardware abstraction layer.  
*   `cgroups.CGroupManager`: Cgroup management interface.  
*   `matcher.Matcher`: Deal matching component.  
*   `ecdsa.PrivateKey`: Ethereum private key.  
*   `YAMLConfig`: Configuration loaded from YAML.  
*   `network.NetworkConfig`: Network configuration.  
*   `DealDestroyer`: Interface for cancelling deal tasks.  
  
**TODOs:**  
  
*   None found in this file.  
  
**Code Summary:**  
  
### Options Configuration  
  
The code defines an `options` struct that holds dependencies for the `salesman` component. These dependencies include logging, storage, resource scheduling, hardware access, Ethereum interaction, cgroup management, deal matching, Ethereum key, configuration, network configuration, and a deal destroyer.  
  
### Option Functions  
  
A series of `With...` functions are provided to configure the `options` struct using functional options pattern. Each function takes a dependency and returns an `Option` function that modifies the `options` struct.  
  
### Validation  
  
The `Validate` method checks if all required dependencies have been provided through the `With...` functions. It returns an error if any dependency is missing. The validation ensures that the component has all the necessary resources before it can operate.  
  
### Option Type  
  
The `Option` type is defined as a function that takes a pointer to the `options` struct and modifies it. This allows for flexible and composable configuration of the component.  
  
# insonmnia/worker/salesman/salesman.go  
## Salesman Component Summary  
  
**Package Name:** `salesman`  
  
**Imports:**  
  
*   `context`  
*   `crypto/ecdsa`  
*   `errors`  
*   `fmt`  
*   `sync`  
*   `time`  
*   `github.com/ethereum/go-ethereum/crypto`  
*   `github.com/mohae/deepcopy`  
*   `github.com/pborman/uuid`  
*   `github.com/sonm-io/core/blockchain`  
*   `github.com/sonm-io/core/insonmnia/cgroups`  
*   `github.com/sonm-io/core/insonmnia/hardware`  
*   `github.com/sonm-io/core/insonmnia/matcher`  
*   `github.com/sonm-io/core/insonmnia/resource`  
*   `github.com/sonm-io/core/insonmnia/state`  
*   `github.com/sonm-io/core/insonmnia/worker/network`  
*   `github.com/sonm-io/core/proto` (as `sonm`)  
*   `github.com/sonm-io/core/util`  
*   `github.com/sonm-io/core/util/multierror`  
*   `github.com/sonm-io/core/util/xconcurrency`  
*   `go.uber.org/zap`  
  
**External Data/Input Sources:**  
  
*   Blockchain API (`blockchain.API`) for interacting with the Ethereum network.  
*   Hardware configuration (`hardware.Hardware`) for resource allocation.  
*   CGroup manager (`cgroups.CGroupManager`) for resource isolation.  
*   Matcher service (`matcher.Matcher`) for deal creation.  
*   Resource scheduler (`resource.Scheduler`) for managing available resources.  
*   Storage (`state.Storage`) for persisting salesman state.  
*   Ethereum private key (`ecdsa.PrivateKey`) for signing transactions.  
*   YAML configuration (`YAMLConfig`) for adjustable parameters (bill periods, timeouts).  
*   Network configuration (`network.Config`) for network settings.  
  
**TODOs:**  
  
*   `//TODO: restore tasks` - Restore tasks functionality is missing.  
*   `//TODO:refactor NetFlags in separqate PR` - NetFlags refactoring is pending.  
*   `//TODO: we will know about closed deal on next iteration for simplicicty, but maybe we can optimize here.` - Optimization of deal closing logic is suggested.  
  
**Code Sections Summary:**  
  
*   **Configuration:** Defines `Config` and `YAMLConfig` structs to hold dependencies (logger, storage, blockchain API, etc.) and configurable parameters (bill periods, timeouts).  
*   **Salesman Struct:** The core `Salesman` struct manages ask plans, deals, cgroups, networks, and synchronization logic. It includes maps for tracking these entities.  
*   **Initialization (`NewSalesman`):** Creates a new `Salesman` instance, initializes dependencies, restores state from storage, and starts background routines.  
*   **Lifecycle Management (`Close`):** Closes the network manager.  
*   **Main Loop (`Run`):** Starts goroutines for waiting for deals and periodic blockchain synchronization.  
*   **State Management:** Functions for creating, removing, and retrieving ask plans and deals. Includes logic for resource allocation, cgroup/network management, and storage persistence.  
*   **Blockchain Synchronization (`syncWithBlockchain`, `syncPlanWithBlockchain`):** Periodically checks the blockchain for deal status, order fulfillment, and performs necessary actions (closing deals, canceling orders).  
*   **Deal/Order Handling:** Functions for creating deals, canceling orders, and handling deal closures.  
*   **Resource Management:** Functions for creating and dropping cgroups and networks associated with ask plans.  
*   **Maintenance Scheduling:** Schedules periodic maintenance tasks.  
*   **Debugging (`DebugDump`):** Provides a snapshot of the salesman's internal state for debugging purposes.  
  
