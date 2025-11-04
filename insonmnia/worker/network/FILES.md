# insonmnia/worker/network/config.go  
**Package / Component Name**    
`network`  
  
---  
  
### Imports  
No external imports are declared in this file.  
  
### External Data / Input Sources  
None specified; the struct is intended to be populated from YAML configuration files (e.g., `remote_qos` key).  
  
### TODOs  
* No TODO comments found.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. Struct Definition – `NetworkConfig`  
The file declares a single Go struct named **`NetworkConfig`**.    
- **Purpose**: Holds configuration data for network-related settings, specifically an optional QOS (Quality of Service) server.  
- **Field**    
  - `RemoteQOS string`: The field stores the address or identifier of a remote QOS server. It is annotated with a YAML tag (`yaml:"remote_qos"`) so that it can be marshalled/unmarshalled from/to YAML files under the key `remote_qos`.    
- **Comments**: Two inline comments explain the role of this field and suggest that users typically will not need to modify it directly.  
  
---  
  
### 2. Usage Context  
This struct is likely part of a larger configuration package where network settings are read from a YAML file, unmarshalled into `NetworkConfig`, and then used by other components (e.g., networking services or deployment scripts).  
  
---  
  
# insonmnia/worker/network/l2tp_config.go  
**Package & Imports**    
```go  
package network  
```  
Imports:  
- `crypto/md5` – MD5 hashing for generating a pool ID.  
- `encoding/hex` – hex encoding of the hash result.  
- `errors`, `fmt`, `net` – standard library helpers.  
- `github.com/docker/go-plugins-helpers/ipam` – request type for IPAM pools.  
- `github.com/docker/go-plugins-helpers/network` – request type for network creation.  
- `github.com/jinzhu/configor` – YAML/JSON config loading.  
  
---  
  
### External Data / Input Sources  
| Source | Description |  
|--------|-------------|  
| `request.Options["config"]` (in `parseOptsIPAM`) | Path to a configuration file that will be loaded into an `l2tpNetworkConfig`. |  
| `request.Options["com.docker.network.generic"]` (in `parseOptsNetwork`) | Nested map containing the same `"config"` key; used when creating a network. |  
  
---  
  
### TODOs  
No explicit TODO comments were found in this file.  
  
---  
  
## Types  
  
### `L2TPConfig`  
A lightweight struct that holds plugin‑level configuration:  
- `Enabled` – whether the L2TP plugin is active.  
- `NetSocketPath`, `IPAMSocketPath` – Unix socket paths for network and IPAM communication.  
- `ConfigDir`, `StatePath` – directories used by the plugin.  
  
### `l2tpNetworkConfig`  
Represents a single L2TP network pool:  
- Core fields: `ID`, `LNSAddr`, `Subnet`.  
- PPP‑related options (`PPPUsername`, `PPPPassword`, MTU/MRU, idle time, connect delay, debug flags, etc.).  
- All fields are tagged for YAML/JSON marshaling and have default values where appropriate.  
  
---  
  
## Methods  
  
### `(o *l2tpNetworkConfig) PoolID() string`  
Computes a unique identifier for the pool by hashing `ID + LNSAddr` with MD5 and returning its hex representation.    
Used as an internal key when registering pools.  
  
### `(o *l2tpNetworkConfig) validate() error`  
Performs basic sanity checks:  
- Parses `LNSAddr` into an IP.  
- Parses `Subnet` into a CIDR block.  
Returns an error if either parsing fails; otherwise returns nil.  
  
---  
  
## Functions  
  
### `parseOptsIPAM(request *ipam.RequestPoolRequest)`  
Extracts the `"config"` option from an IPAM request, loads it with `configor`, validates it, and returns the resulting `l2tpNetworkConfig`.  
  
### `parseOptsNetwork(request *network.CreateNetworkRequest)`  
Similar to `parseOptsIPAM` but expects a nested map under key `"com.docker.network.generic"`.    
It pulls the same `"config"` path, loads, validates, and returns an `l2tpNetworkConfig`.  
  
### `getAddrFromCIDR(cidr string) (string, error)`  
Convenience helper that parses a CIDR string into an IP address and returns it as a string.  
  
---  
  
**<end_of_output>**  
  
# insonmnia/worker/network/l2tp_ipam.go  
**Package / Component**  
  
- **Name:** `network`  
- **Imports:**  
  ```go  
  import (  
      "context"  
      "errors"  
      "fmt"  
      "io/ioutil"  
      "net"  
      "os"  
      "os/exec"  
      "time"  
  
      "github.com/docker/go-plugins-helpers/ipam"  
      log "github.com/noxiouz/zapctx/ctxlog"  
      "go.uber.org/zap"  
  )  
  ```  
  
**External Data / Input Sources**  
  
| Source | Description |  
|--------|-------------|  
| `ipam.RequestPoolRequest` | Request data for creating a new IPAM pool |  
| `ipam.RequestAddressRequest` | Request data for allocating an address in the pool |  
| `ipam.ReleasePoolRequest` | Request data for releasing a pool |  
| `ipam.ReleaseAddressRequest` | Request data for releasing an address |  
| `l2tpState`, `l2tpNetwork`, `l2tpEndpoint` | State and network structures defined elsewhere in the package |  
| External commands: `xl2tpd-control` | Used to add, connect, and disconnect xl2tp connections |  
  
**TODO Comments**  
  
- No explicit TODO markers found; all functionality is implemented.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. `IPAMDriver` struct  
```go  
type IPAMDriver struct {  
    *l2tpState  
    counter int  
    logger  *zap.SugaredLogger  
}  
```  
Holds a reference to the overall state, a simple counter, and a logger for tracing.  
  
### 2. `NewIPAMDriver`  
Creates a new driver instance, initializing the logger with context information.  
  
### 3. `RequestPool`  
Handles an IPAM pool creation request:  
- Logs receipt of the request.  
- Locks the internal mutex, parses options via `parseOptsIPAM`.  
- Creates a new network (`newL2tpNetwork`), sets it up, and adds it to state.  
- Returns a response containing the new pool ID and subnet.  
  
### 4. `RequestAddress`  
Handles an IPAM address allocation request:  
- Logs receipt of the request.  
- Locks mutex, retrieves the target network.  
- If the network needs a gateway, marks it as allocated and returns immediately.  
- Creates a new endpoint (`NewL2TPEndpoint`), sets it up, writes PPP options to disk, runs external commands to add and connect xl2tp connections.  
- Retrieves the assigned CIDR via `getAssignedCIDR`, stores it in the endpoint, and returns an address response.  
  
### 5. `ReleasePool`  
Releases a pool:  
- Logs receipt of the request.  
- Retrieves the network, removes its endpoint with `removeEndpoint`, then deletes the network from state.  
  
### 6. `ReleaseAddress`  
Placeholder for future implementation; currently just logs the request and returns nil.  
  
### 7. `GetCapabilities`  
Returns static capabilities information (currently only indicates that MAC addresses are not required).  
  
### 8. `GetDefaultAddressSpaces`  
Returns an empty address space response; placeholder for future expansion.  
  
### 9. `getAssignedCIDR`  
Sleeps 7 seconds, enumerates network interfaces, finds the one matching a given device name, and returns its first IP address as a CIDR string.  
  
### 10. `removeEndpoint`  
Disconnects an xl2tp connection via external command, removes the PPP options file, and cleans up the endpoint.  
  
---  
  
All functions use structured logging (`zap`) for debugging and traceability. The driver orchestrates network creation, endpoint configuration, and IP assignment through a combination of internal state manipulation and external system commands.  
  
# insonmnia/worker/network/l2tp_network.go  
**Package / Component**    
`network`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"fmt"  
  
	"github.com/docker/go-plugins-helpers/network"  
	log "github.com/noxiouz/zapctx/ctxlog"  
	"go.uber.org/zap"  
)  
```  
* `context`, `fmt` – standard Go packages    
* `github.com/docker/go-plugins-helpers/network` – plugin‑helper types and interfaces    
* `github.com/noxiouz/zapctx/ctxlog` – logger helper (aliased as `log`)    
* `go.uber.org/zap` – Uber’s Zap logging library  
  
---  
  
### External data / input sources  
The driver interacts with the following external structures:  
| Function | External type used |  
|----------|---------------------|  
| `GetCapabilities()` | `network.CapabilitiesResponse` |  
| `CreateNetwork(request *network.CreateNetworkRequest)` | `network.CreateNetworkRequest` |  
| `CreateEndpoint(request *network.CreateEndpointRequest)` | `network.CreateEndpointRequest`, returns `*network.CreateEndpointResponse` |  
| `Join(request *network.JoinRequest)` | `network.JoinRequest`, returns `*network.JoinResponse` |  
| `Leave(request *network.LeaveRequest)` | `network.LeaveRequest` |  
| `EndpointInfo(request *network.InfoRequest)` | `network.InfoRequest`, returns `*network.InfoResponse` |  
| `AllocateNetwork(request *network.AllocateNetworkRequest)` | `network.AllocateNetworkRequest`, returns `*network.AllocateNetworkResponse` |  
| `DeleteNetwork(request *network.DeleteNetworkRequest)` | `network.DeleteNetworkRequest` |  
| `FreeNetwork(request *network.FreeNetworkRequest)` | `network.FreeNetworkRequest` |  
| `DeleteEndpoint(request *network.DeleteEndpointRequest)` | `network.DeleteEndpointRequest` |  
| `DiscoverNew(request *network.DiscoveryNotification)` | `network.DiscoveryNotification` |  
| `DiscoverDelete(request *network.DiscoveryNotification)` | `network.DiscoveryNotification` |  
| `ProgramExternalConnectivity(request *network.ProgramExternalConnectivityRequest)` | `network.ProgramExternalConnectivityRequest` |  
| `RevokeExternalConnectivity(request *network.RevokeExternalConnectivityRequest)` | `network.RevokeExternalConnectivityRequest` |  
  
---  
  
### TODO list  
No explicit `TODO:` comments are present in the file, but several methods still return `nil, nil`.    
These should be implemented later:  
* `AllocateNetwork`  
* `DeleteNetwork`  
* `FreeNetwork`  
* `DeleteEndpoint`  
* `DiscoverNew`  
* `DiscoverDelete`  
* `ProgramExternalConnectivity`  
* `RevokeExternalConnectivity`  
  
---  
  
## Code Summary  
  
### L2TPNetworkDriver struct  
```go  
type L2TPNetworkDriver struct {  
	*l2tpState  
	logger *zap.SugaredLogger  
}  
```  
Holds a pointer to the internal state (`l2tpState`) and a Sugared logger for debug output.  
  
### NewL2TPDriver constructor  
Creates a new driver instance, wiring the provided `state` and initializing the logger with context `"source": "l2tp/network"`.  
  
### GetCapabilities  
Logs receipt of the request and returns a minimal capabilities response indicating local connectivity scope.  
  
### CreateNetwork  
* Locks the internal mutex, parses options from the incoming request (`parseOptsNetwork`), retrieves the target network by pool ID, sets its ID, registers an alias, logs success, and unlocks.  
* The method currently ends with `return nil`, meaning further logic (e.g., persisting to storage) is pending.  
  
### CreateEndpoint  
* Locks, fetches the network, validates that an endpoint exists and has no existing ID, assigns the new endpoint ID from the request, and returns a placeholder response.    
* Future work: populate the returned `CreateEndpointResponse` with actual data.  
  
### Join  
* Logs receipt, locks, retrieves the target network, logs joined endpoint details, and builds a `JoinResponse` containing interface name and static routes.  
* The route type is hard‑coded to 1; this may need refinement later.  
  
### Leave  
Placeholder – currently only logs receipt and returns nil.  
  
### EndpointInfo  
* Locks, fetches the network, and constructs an `InfoResponse` map with IP, connection name, and device name from the endpoint.    
* Future work: populate additional fields if needed.  
  
### AllocateNetwork  
Logs receipt; placeholder for future implementation.  
  
### DeleteNetwork  
Logs receipt; placeholder.  
  
### FreeNetwork  
Logs receipt; placeholder.  
  
### DeleteEndpoint  
Logs receipt; placeholder.  
  
### DiscoverNew / DiscoverDelete  
Both log receipt of discovery notifications and return nil – to be expanded later.  
  
### ProgramExternalConnectivity / RevokeExternalConnectivity  
Log receipt of external connectivity requests; currently no implementation beyond the log.  
  
---  
  
All methods follow a similar pattern: log, lock, defer unlock/sync, perform operation, log result.    
The driver is ready for integration into a larger plugin system once the remaining stubs are fleshed out.  
  
# insonmnia/worker/network/l2tp_state.go  
**Package name**    
`network`  
  
**Imports**  
  
| Package | Purpose |  
|---------|---------|  
| `bytes` | Buffer handling for PPP config generation |  
| `context` | Context passed to the store constructor |  
| `encoding/json` | Marshal/unmarshal state data |  
| `fmt` | String formatting and logging |  
| `sync` | Mutex for concurrent access to state |  
| `github.com/docker/libkv` | Store abstraction |  
| `github.com/docker/libkv/store` | Backend interface |  
| `github.com/docker/libkv/store/boltdb` | BoltDB backend registration |  
| `log "github.com/noxiouz/zapctx/ctxlog"` | Structured logger alias |  
| `go.uber.org/zap` | Zap logging integration |  
  
---  
  
### External data / input sources  
* Constant directory for PPP options: `pppOptsDir = "/etc/ppp/"`.  
* BoltDB backend with bucket `"sonm_l2tp_driver_state"`.  
* Configuration values are read from a `l2tpNetworkConfig` (type defined elsewhere in the package).  
  
---  
  
### TODO list  
No explicit `TODO:` comments were found in this file.  
  
---  
  
## Major code parts  
  
### 1. `l2tpState`  
Represents the overall state of L2TP networks and provides persistence via libkv/boltdb.  
  
* **Fields**    
  * `mu sync.Mutex` – protects concurrent access.    
  * `Aliases map[string]string` – maps network IDs to internal identifiers.    
  * `Networks map[string]*l2tpNetwork` – collection of network objects.    
  * `storage store.Store` – BoltDB-backed key/value store.    
  * `logger *zap.SugaredLogger` – structured logger.  
  
* **Constructor** `newL2TPNetworkState(ctx context.Context, path string)`    
  Registers the BoltDB backend, creates a new store with bucket `"sonm_l2tp_driver_state"`, initializes maps and logger, then loads existing state from storage.  
  
* **Persistence helpers**    
  * `sync()` – marshals the whole struct to JSON and writes it under key `"state"` in the store.    
  * `load()` – reads the stored JSON, unmarshals into the struct, validates that each network has an endpoint, logs a message for each loaded network.  
  
* **Network management**    
  * `AddNetwork(netID string, netInfo *l2tpNetwork)` – registers a new network and its alias.    
  * `AddNetworkAlias(netID, alias string)` – adds an additional alias pointing to the same internal ID.    
  * `RemoveNetwork(netID string)` – removes a network by alias, cleans up aliases that point to it.    
  * `GetNetwork(netID string) (*l2tpNetwork, error)` – retrieves a network by its alias.  
  
---  
  
### 2. `l2tpNetwork`  
Represents an individual L2TP network configuration and holds a reference to its endpoint.  
  
* **Fields**    
  * `ID`, `PoolID` – identifiers for the network.    
  * `Count int` – usage counter.    
  * `NetworkOpts *l2tpNetworkConfig` – pointer to shared config values.    
  * `Endpoint *l2tpEndpoint` – endpoint that will be used by this network.    
  * `NeedsGateway bool` – flag for gateway requirement.  
  
* **Constructor** `newL2tpNetwork(opts *l2tpNetworkConfig)` – creates a new instance with default values and sets `NeedsGateway=true`.  
  
* **Lifecycle helpers**    
  * `Setup()` – currently only assigns the pool ID from config.    
  * `ConnInc()` – increments the usage counter.  
  
---  
  
### 3. `l2tpEndpoint`  
Represents an endpoint that will be used by a network to establish PPP connections.  
  
* **Fields**    
  * `ID`, `Name`, `ConnName` – identifiers for the endpoint and its connection.    
  * `PPPOptFile`, `PPPDevName`, `AssignedCIDR`, `AssignedIP` – PPP configuration details.    
  * `NetworkOpts *l2tpNetworkConfig` – shared config reference.  
  
* **Constructor** `NewL2TPEndpoint(netInfo *l2tpNetwork)` – builds an endpoint with a name derived from the network’s counter and pool ID, and copies the network options.  
  
* **Configuration helpers**    
  * `setup()` – sets connection name, PPP device name (truncated to 15 chars), and PPP option file path.    
  * `GetPppConfig() string` – builds a PPP configuration string from all enabled flags in `NetworkOpts`.    
  * `GetXl2tpConfig() []string` – returns two XL2TP‑specific config lines (`lns`, `pppoptfile`) for use by the driver.  
  
---  
  
# insonmnia/worker/network/l2tp_tuner.go  
# Package `network`  
  
## Imports  
```go  
import (  
	"context"  
	"errors"  
	"fmt"  
	"io/ioutil"  
	"net"  
	"os"  
	"syscall"  
  
	"github.com/docker/docker/api/types"  
	"github.com/docker/docker/api/types/container"  
	"github.com/docker/docker/api/types/network"  
	"github.com/docker/docker/client"  
	"github.com/docker/go-plugins-helpers/ipam"  
	netDriver "github.com/docker/go-plugins-helpers/network"  
	log "github.com/noxiouz/zapctx/ctxlog"  
	"github.com/sonm-io/core/insonmnia/structs"  
	"go.uber.org/zap"  
)  
```  
  
## External data / input sources  
| Source | Description |  
|--------|-------------|  
| `cfg.ConfigDir` | Directory where configuration files are stored (created with `os.MkdirAll`). |  
| `cfg.StatePath` | Path used by `newL2TPNetworkState(ctx, cfg.StatePath)` to initialise network state. |  
| Unix sockets | Two sockets: one at `t.cfg.NetSocketPath` for the L2TP network driver and another at `t.cfg.IPAMSocketPath` for the IPAM driver. |  
| Docker client | `client.NewEnvClient()` creates a Docker client used throughout the tuner. |  
  
## TODOs  
- **GenerateInvitation** – currently returns an error placeholder; needs implementation.  
  
---  
  
## Overview of the main type  
  
```go  
type L2TPTuner struct {  
	cfg        *L2TPConfig  
	cli        *client.Client  
	netDriver  *L2TPNetworkDriver  
	ipamDriver *IPAMDriver  
}  
```  
  
`L2TPTuner` holds configuration, a Docker client, and two driver instances (network & IPAM). It exposes methods to create the tuner, run listeners, tune networks, and clean up.  
  
---  
  
## `NewL2TPTuner`  
*Creates the tuner instance.*  
  
1. Creates the config directory (`os.MkdirAll`).    
2. Instantiates a Docker client with `client.NewEnvClient()`.    
3. Loads network state via `newL2TPNetworkState(ctx, cfg.StatePath)`.    
4. Builds the struct and immediately calls `tuner.Run(ctx)` to start listeners.  
  
---  
  
## `Run`  
*Initialises Unix socket listeners for both drivers.*  
  
- Unlinks any existing sockets at `NetSocketPath` and `IPAMSocketPath`.  
- Creates a listener on each path with `net.Listen("unix", …)`.    
- Wraps the drivers in handlers (`netDriver.NewHandler`, `ipam.NewHandler`).    
- Starts three goroutines:  
  * One to close listeners when context is cancelled.  
  * One to serve the IPAM driver.  
  * One to serve the network driver.  
  
---  
  
## `Tune`  
*Creates a Docker network and writes its configuration.*  
  
1. Calls `writeConfig` to persist options for the given network ID.    
2. Builds a `types.NetworkCreate` struct with driver names `"l2tp_net"` and `"l2tp_ipam"`.    
3. Invokes `t.cli.NetworkCreate(ctx, net.NetID, createOpts)` to create the network.    
4. Populates `netCfg.EndpointsConfig` if missing, adding endpoint settings for the new network.    
5. Returns an `L2TPCleaner` that can later remove the network and its config file.  
  
---  
  
## `GetCleaner`  
*Retrieves a cleaner for an existing network ID.*  
  
- Checks that the ID exists in `t.netDriver.Networks`.    
- Builds a path to the stored config file and returns an `L2TPCleaner`.  
  
---  
  
## Helper: `writeConfig`  
Writes key/value options into a plain‑text file.  
  
```go  
func (t *L2TPTuner) writeConfig(netID string, opts map[string]string) (string, error) {  
	var data string  
	for k, v := range opts {  
		data += fmt.Sprintf("%s: %s\n", k, v)  
	}  
	path := t.cfg.ConfigDir + "/" + netID  
	return path, ioutil.WriteFile(path, []byte(data), 0700)  
}  
```  
  
---  
  
## `L2TPCleaner`  
*Handles cleanup of a network.*  
  
```go  
type L2TPCleaner struct {  
	ctx        context.Context  
	cli        *client.Client  
	networkID  string  
	configPath string  
}  
  
func (t *L2TPCleaner) Close() error {  
	log.G(t.ctx).Info("closing l2tp driver")  
	if err := t.cli.NetworkRemove(t.ctx, t.networkID); err != nil {  
		return err  
	}  
	return os.Remove(t.configPath)  
}  
```  
  
`Close()` removes the Docker network and deletes its config file.  
  
---  
  
**<end_of_output>**  
  
# insonmnia/worker/network/manager.go  
# Package `network`  
  
## Imports  
```go  
import (  
	"context"  
	"fmt"  
	"net/url"  
	"strings"  
	"sync"  
  
	"github.com/docker/docker/api/types"  
	"github.com/docker/docker/api/types/filters"  
	"github.com/docker/docker/client"  
	"github.com/sonm-io/core/insonmnia/worker/network/tc"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util/multierror"  
	"github.com/sonm-io/core/util/xgrpc"  
	"go.uber.org/zap"  
)  
```  
  
## External data / input sources  
| Source | Purpose |  
|--------|---------|  
| Docker client (`github.com/docker/docker/client`) | Create, list and remove Docker networks |  
| xgrpc client (`github.com/sonm-io/core/util/xgrpc`) | Remote QoS client for remote network manager |  
| `tc.TC` (`github.com/sonm-io/core/insonmnia/worker/network/tc`) | Local network manager implementation |  
| `zap.SugaredLogger` (`go.uber.org/zap`) | Logging throughout the package |  
  
## TODO list  
No explicit `TODO:` comments were found in this file.  
  
---  
  
# Summary of major code parts  
  
## Constants & configuration  
```go  
const (  
	networkPrefix       = "sonm"  
	networkIfbPrefix    = "ifb-"  
	driverBridge        = "bridge"  
	tagSonmNetwork      = "com.sonm.network"  
	tagSonmNetworkAlias = "com.sonm.network.alias"  
	maxLinkNameLen      = 12  
)  
```  
Defines prefixes, tags and the maximum length for interface names.  
  
## Action abstraction & queue  
* `Action` – interface with `Execute` and `Rollback`.  
* `ActionQueue` – a thread‑safe stack of actions that can be executed in order and rolled back if needed.  
  * `NewActionQueue`, `Execute`, `Rollback`, internal helper `rollback` and `pop`.  
  
## Core data structures  
| Type | Description |  
|------|-------------|  
| `Network` | Holds ID, name, alias and bandwidth limits. |  
| `CreateNetworkRequest` | Parameters for creating a network (ID + limits). |  
| `PruneRequest` | List of IDs to prune. |  
| `PruneReply` | Result map from pruning operation. |  
| `NetworkManagerConfig` | Configuration for the manager (remote address, Docker client, logger). |  
| `networkManager` interface | Methods that a concrete manager must implement (`Init`, `Close`, `NewActions`). |  
| `NetworkManager` | Holds an implementation of `networkManager` and shared resources. |  
  
## Options pattern  
* `options` struct – holds the chosen manager, Docker client and logger.  
* `newOptions()` – creates default options (Docker env client + local manager).  
* `Option` type – functional option for configuring `options`.  
* `WithRemote(uri string)` – parses a remote QoS URI and sets up a remote network manager if needed.  
* `WithLog(log *zap.SugaredLogger)` – injects a logger.  
  
## Manager construction & lifecycle  
```go  
func NewNetworkManager(options ...Option) (*NetworkManager, error)  
```  
Creates the manager, applies all options, initializes the underlying manager and returns it.  
  
### Create network  
```go  
func (m *NetworkManager) CreateNetwork(ctx context.Context, request *CreateNetworkRequest) (*Network, error)  
```  
* Builds a network name from prefix + ID.  
* Truncates to OS‑required length (`truncLinkName`).  
* Creates an action queue and executes all actions returned by the underlying manager.  
  
### Remove & prune  
```go  
func (m *NetworkManager) RemoveNetwork(network *Network) error  
func (m *NetworkManager) Prune(ctx context.Context, request *PruneRequest) (*PruneReply, error)  
```  
* `RemoveNetwork` simply rolls back all actions of the underlying manager.  
* `Prune` builds a set of network names from IDs, lists Docker networks with the SONM tag, removes each and returns a map of results.  
  
### Close  
```go  
func (m *NetworkManager) Close() error  
```  
Delegates to the underlying manager’s `Close`.  
  
## Docker network action implementation  
* `DockerNetworkCreateAction` – concrete action that creates a Docker network.  
  * `Execute(ctx)` – calls Docker client’s `NetworkCreate`, sets the ID, and logs errors.  
  * `Rollback()` – removes the created network.  
  * `isErrNetworkAlreadyExists(err error) bool` – helper to detect duplicate‑network errors.  
  
## Utility functions  
```go  
func truncLinkName(v string) (string, bool)  
```  
Truncates a name to `maxLinkNameLen` and indicates whether it was truncated.  
  
## Local manager implementation  
* `localNetworkManager` – holds a `tc.TC` instance and Docker client.  
  * `Close()` – closes the underlying TC connection.  
  
---  
  
All of these pieces together provide a high‑level abstraction for creating, removing, pruning and managing Docker networks with optional remote QoS support.  
  
# insonmnia/worker/network/manager_linux.go  
# Package Overview    
**Package name:** `network`    
  
This file implements a set of actions for managing Linux network interfaces and traffic control (TC) disciplines. It defines two concrete action types (`NetworkAliasAction`, `TBFShapingAction`, `HTBShapingAction`) that can be used by a local manager or a remote QOS server to create, alias, and shape traffic on Docker networks.  
  
---  
  
## Imports    
| Import | Purpose |  
|--------|---------|  
| `context` | Provides the context type for action execution. |  
| `fmt` | String formatting and error messages. |  
| `syscall` | System call constants (used in error handling). |  
| `github.com/sonm-io/core/insonmnia/worker/network/tc` | Traffic‑control helper types (`TC`, `QDisc`, etc.). |  
| `github.com/sonm-io/core/proto` | Protocol buffer definitions for QOS requests/responses. |  
| `github.com/sonm-io/core/util/multierror` | Aggregates multiple errors into a single error value. |  
| `github.com/vishvananda/netlink` | Low‑level netlink interface for manipulating Linux network links. |  
  
---  
  
## External Data / Input Sources    
* **Network struct** – referenced in all actions; holds fields such as `Name`, `Alias`, `RateLimitIngress`, `RateLimitEgress`.    
* **sonm.QOSServer** – the file implements this interface via `RemoteQOS`.    
* **Proto request/response types** (`QOSSetAliasRequest`, `QOSAddHTBShapingRequest`, etc.) used by RemoteQOS methods.    
  
---  
  
## TODOs    
No explicit `TODO:` comments are present in the code; all functionality is already implemented.  
  
---  
  
# Major Code Parts  
  
## 1. Network Alias Action  
```go  
type NetworkAliasAction struct {  
    Network *Network  
}  
```  
* **Execute** – looks up a link by name, then sets an alias on it using `netlink.LinkSetAlias`.    
* **Rollback** – currently does nothing; the alias will be removed automatically when the interface is deleted.  
  
## 2. TBF Shaping Action  
```go  
type TBFShapingAction struct {  
    Network *Network  
    tc      tc.TC  
    rootQDiscHandle tc.QDisc  
}  
```  
* **Execute** –    
  - Adds a root `TFBQDisc` with rate/latency settings.    
  - Adds an egress `Ingress` discipline and a default filter (`U32`).    
  - Stores the created root QDisc handle for later rollback.    
* **Rollback** – deletes the stored root QDisc if present.  
  
## 3. HTB Shaping Action  
```go  
type HTBShapingAction struct {  
    Network *Network  
    tc      tc.TC  
    rootQDiscHandle tc.QDisc  
    ifbLink netlink.Link  
}  
```  
* **Execute** –    
  - Initializes IFB, adds a root `HTBQDisc`, deletes any existing discipline, and stores the handle.    
  - Creates an HTB class and leaf `PfifoQDisc`.    
  - Adds filters for ingress/egress shaping.    
  - Configures traffic mirroring via an IFB link (`ifbLink`) that is created with `newIFBLink()`.    
* **Rollback** – deletes the IFB link, then removes the root QDisc if present.  
  
## 4. Helper Functions  
```go  
func (m *HTBShapingAction) newIFBLink() *netlink.Ifb { ... }  
func NewIFBLink(name string) *netlink.Ifb { ... }  
```  
Creates a new IFB link with a name derived from the network alias.  
  
## 5. Local Network Manager  
```go  
func (m *localNetworkManager) Init() error { ... }  
func (m *localNetworkManager) NewActions(network *Network) []Action { ... }  
```  
* `Init` – flushes IFB, creates a default TC instance and stores it in the manager.    
* `NewActions` – returns a slice of actions to be executed for a given network: Docker creation, aliasing, and HTB shaping.  
  
## 6. Remote QOS Server  
```go  
type RemoteQOS struct { tc tc.TC }  
func NewRemoteQOS() (*RemoteQOS, error) { ... }  
```  
* Implements `sonm.QOSServer` via methods:  
  - `SetAlias`: executes a `NetworkAliasAction`.    
  - `AddHTBShaping`: executes an `HTBShapingAction`.    
  - `RemoveHTBShaping`: prepares and rolls back an HTB shaping action.    
  - `Flush`: flushes IFB.  
  
All methods use the underlying TC instance (`m.tc`) to add/delete QDiscs, classes, filters, etc., and return appropriate protocol buffer responses.  
  
---  
  
# insonmnia/worker/network/manager_nonlinux.go  
**Package / Component**    
`network`  
  
---  
  
### Imports  
```go  
import (  
    "context"  
    "errors"  
  
    "github.com/sonm-io/core/insonmnia/worker/network/tc"  
    "github.com/sonm-io/core/proto"  
)  
```  
* `context` – standard Go context package    
* `errors` – standard Go errors package    
* `tc` – local worker network types (QDisc, Class, Filter)    
* `proto` – core protocol definitions used by the QOS server  
  
---  
  
### External Data / Input Sources  
| Source | Description |  
|--------|-------------|  
| `tc.QDisc`, `tc.Class`, `tc.Filter` | Network configuration objects handled by the local TC implementation. |  
| `sonm.QOSSetAliasRequest`, `sonm.QOSAddHTBShapingRequest`, `sonm.QOSRemoveHTBShapingRequest`, `sonm.QOSFlushRequest` | Requests sent to a QOS server (implemented by `nilQOS`). |  
  
---  
  
### TODOs  
* All methods of the `TC` struct currently return `ErrUnsupportedPlatform`.    
  * Implement real logic for adding/deleting qdiscs, classes, and filters.    
* The `Close()` method is a stub – add cleanup logic if needed.    
* `NewRemoteQOS()` returns a dummy implementation; replace with a proper remote QOS server.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. `TC` struct  
A lightweight placeholder for the local network controller.    
Methods:  
- `QDiscAdd`, `QDiscDel`: add/delete qdisc objects.  
- `ClassAdd`, `ClassDel`: add/delete class objects.  
- `FilterAdd`, `FilterDel`: add/delete filter objects.  
- `Close`: currently a no‑op; intended to release resources.  
  
All methods return the global error `ErrUnsupportedPlatform` until implemented.  
  
### 2. `localNetworkManager.Init`  
Initializes the manager by assigning a new `TC` instance to its internal field:  
```go  
func (m *localNetworkManager) Init() error {  
    m.tc = &TC{}  
    return nil  
}  
```  
  
### 3. `localNetworkManager.NewActions`  
Creates an action slice for network creation, currently containing only a Docker‑based action:  
```go  
func (m *localNetworkManager) NewActions(network *Network) []Action {  
    return []Action{  
        &DockerNetworkCreateAction{  
            DockerClient: m.dockerClient,  
            Network:      network,  
        },  
    }  
}  
```  
  
### 4. `nilQOS` struct  
A stub implementation of the QOS server interface, providing:  
- `SetAlias`  
- `AddHTBShaping`  
- `RemoveHTBShaping`  
- `Flush`  
  
Each method returns `ErrUnsupportedPlatform`; they need real logic.  
  
### 5. `NewRemoteQOS`  
Factory function returning a new `nilQOS` instance as a `sonm.QOSServer`.  
  
---  
  
**End of output**  
  
# insonmnia/worker/network/manager_remote.go  
**Package & Imports**    
- **Package name:** `network`    
- **Imports:**  
  ```go  
  import (  
      "context"  
      "time"  
  
      "github.com/docker/docker/client"  
      "github.com/sonm-io/core/proto"  
  )  
  ```  
  
---  
  
### External Data / Input Sources    
The file interacts with two external clients:  
1. `sonm.QOSClient` – used to set aliases, add HTB shaping and flush QOS requests.  
2. Docker client (`*client.Client`) – passed into the network creation action.  
  
All actions operate on a `Network` struct (defined elsewhere in the same package) that contains fields such as `Name`, `Alias`, `RateLimitEgress`, and `RateLimitIngress`.  
  
---  
  
### TODO Comments    
No explicit `// TODO` comments are present in this file.    
  
---  
  
## Summary of Major Code Parts  
  
### 1. `RemoteNetworkAliasAction`  
- **Purpose:** Sets an alias for a network link.  
- **Fields:**  
  - `Client sonm.QOSClient`  
  - `Network *Network`  
- **Execute:** Calls `SetAlias` on the QOS client with a timeout of 5 s, passing the network name and alias.  
- **Rollback:** Currently a no‑op; comment indicates removal will be handled automatically.  
  
### 2. `RemoteHTBShapingAction`  
- **Purpose:** Adds HTB shaping parameters to a network link.  
- **Fields:**  
  - `Client sonm.QOSClient`  
  - `Network *Network`  
- **Execute:** Calls `AddHTBShaping` with the network name, alias, egress and ingress rate limits.  
- **Rollback:** Removes the HTB shaping via `RemoveHTBShaping`, again using a 5 s timeout.  
  
### 3. `remoteNetworkManager`  
- **Purpose:** Implements the `networkManager` interface for remote operations.  
- **Fields:**  
  - `client sonm.QOSClient`  
  - `dockerClient *client.Client`  
- **Init:** Flushes QOS state with a 5 s timeout.  
- **Close:** Stub returning nil (no cleanup logic yet).  
- **NewActions:** Returns a slice of actions to be executed for a given network:  
  1. Docker network creation (`DockerNetworkCreateAction`).  
  2. Alias setting (`RemoteNetworkAliasAction`).  
  3. HTB shaping (`RemoteHTBShapingAction`).  
  
The `var _ networkManager = &remoteNetworkManager{}` line guarantees that `remoteNetworkManager` satisfies the `networkManager` interface at compile time.  
  
---  
  
# insonmnia/worker/network/tinc_config.go  
**Package/Component Name**    
`network`  
  
---  
  
### Imports  
No external imports are declared in this file; the package relies solely on Go's standard library and any other files within the `network` package.  
  
### External Data / Input Sources  
- YAML configuration files that provide values for the fields of `TincNetworkConfig`.    
  - `enabled`: whether the Tinc network is active.    
  - `config_dir`: directory where Tinc stores its configuration (default `/tinc`).    
  - `docker_net_plugin_dir`: socket path for Docker networking plugin (default `/run/docker/plugins/tinc/tinc.sock`).    
  - `docker_ipam_plugin_dir`: socket path for Docker IPAM plugin (default `/run/docker/plugins/tincipam/tincipam.sock`).    
  - `docker_image`: Docker image used by Tinc (default `sonm/tinc`).    
  - `state_path`: file system location where the network state is persisted (default `/var/lib/sonm/tinc_network_state`).  
  
### TODOs  
No explicit TODO comments are present in this snippet.  
  
---  
  
## Summary of Major Code Parts  
  
#### Struct Definition: `TincNetworkConfig`  
- **Purpose**: Holds configuration parameters for a Tinc-based Docker networking setup.    
- **Fields**:  
  - `Enabled`: Boolean flag to enable/disable the network.  
  - `ConfigDir`: Path where Tinc writes its config files; defaults to `/tinc`.  
  - `DockerNetPluginSockPath`: Socket path used by the Docker networking plugin; defaults to `/run/docker/plugins/tinc/tinc.sock`.  
  - `DockerIPAMPluginSockPath`: Socket path for the IPAM (IP Address Management) plugin; defaults to `/run/docker/plugins/tincipam/tincipam.sock`.  
  - `DockerImage`: Name of the Docker image that runs Tinc; defaults to `sonm/tinc`.  
  - `StatePath`: Path where network state is stored; defaults to `/var/lib/sonm/tinc_network_state`.  
  
The struct tags (`yaml:"..."`) indicate how values are marshalled/unmarshalled from YAML configuration files, making this struct a central piece for configuring the Tinc networking component.  
  
---  
  
# insonmnia/worker/network/tinc_driver.go  
**Package / Component**    
`network`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"fmt"  
	"net"  
	"strings"  
  
	"github.com/docker/docker/client"  
	"github.com/docker/go-plugins-helpers/network"  
	log "github.com/noxiouz/zapctx/ctxlog"  
	"github.com/pborman/uuid"  
	"github.com/sonm-io/core/insonmnia/structs"  
	"github.com/sonm-io/core/proto"  
	"go.uber.org/zap"  
)  
```  
  
---  
  
### External data / input sources  
| Source | Description |  
|--------|-------------|  
| `TincNetworkConfig` | Configuration passed to `NewTinc`; contains options for the network driver (not defined in this file). |  
| `client.Client` | Docker client used to interact with Docker API. |  
| `network.CapabilitiesResponse`, `CreateNetworkRequest`, etc. | Types from `github.com/docker/go-plugins-helpers/network`. |  
  
---  
  
### TODOs  
* **GenerateInvitation** – “Check this” comment indicates that the invitation generation logic may need further refinement.  
  
---  
  
## Summary of major code parts  
  
#### 1. `NewTinc`  
Creates a new network driver instance and an IPAM driver, wiring them together with a logger.  
```go  
func NewTinc(ctx context.Context, client *client.Client, config *TincNetworkConfig) (*TincNetworkDriver, *TincIPAMDriver, error)  
```  
* Calls `newTincNetworkState` to build the internal state.    
* Instantiates `TincNetworkDriver`, attaches a logger, and returns it along with an IPAM driver.  
  
#### 2. Data structures  
| Type | Purpose |  
|------|---------|  
| `TincNetwork` | Holds network metadata (NodeID, DockerID, IP pool, invitation string, bridge flag, etc.) plus client & logger references. |  
| `TincNetworkDriver` | Embeds a pointer to the internal state (`*TincNetworkState`) and owns a SugaredLogger for logging. |  
  
#### 3. Capability handling  
```go  
func (t *TincNetworkDriver) GetCapabilities() (*network.CapabilitiesResponse, error)  
```  
Returns static capabilities indicating local scope.  
  
#### 4. Network lifecycle methods  
| Method | Role |  
|--------|------|  
| `CreateNetwork` | Creates a network from options, sets DockerID, joins or initializes it, then syncs state. |  
| `DeleteNetwork` | Deletes a network by Docker ID and shuts it down. |  
| `FreeNetwork` | Placeholder for freeing resources (currently no logic). |  
  
#### 5. Endpoint handling  
| Method | Role |  
|--------|------|  
| `CreateEndpoint` | Creates an endpoint on the network, starts it with local address. |  
| `DeleteEndpoint` | Stops and removes an endpoint from internal map. |  
| `EndpointInfo` | Returns a simple key/value map (placeholder). |  
  
#### 6. Discovery & connectivity  
* `Join`, `Leave`, `DiscoverNew`, `DiscoverDelete`, `ProgramExternalConnectivity`, `RevokeExternalConnectivity` – all log the request, perform minimal logic or return placeholders.  
  
#### 7. Network lookup helpers  
| Method | Role |  
|--------|------|  
| `popNetwork` | Removes a network from internal map by ID and syncs state. |  
| `netByOptions`, `netByDockerID` – used in several methods (implementation not shown). |  
  
#### 8. Invitation generation  
```go  
func (t *TincNetworkDriver) GenerateInvitation(NodeID string) (*structs.NetworkSpec, error)  
```  
* Locks the driver state, retrieves a network by NodeID, generates an invitation ID using `uuid.New()`, calls `n.Invite` to create an invitation string, and returns a `structs.NetworkSpec` populated with that data.    
* TODO comment indicates further checks may be needed.  
  
---  
  
All methods use the SugaredLogger (`log.S(ctx)`) for structured logging throughout the driver’s lifecycle.  
  
# insonmnia/worker/network/tinc_ipam.go  
**Package / Component**    
`network`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"errors"  
	"fmt"  
	"math/rand"  
	"net"  
  
	"github.com/docker/go-plugins-helpers/ipam"  
  
	log "github.com/noxiouz/zapctx/ctxlog"  
	"go.uber.org/zap"  
)  
```  
* `context`, `errors`, `fmt`, `math/rand`, `net` – standard Go packages    
* `ipam` – Docker IPAM plugin helpers (provides request/response types)    
* `zapctx/ctxlog` – logging helper, aliased as `log`    
* `go.uber.org/zap` – Uber’s Zap logger  
  
---  
  
### External data / input sources  
| Function | Source of data |  
|----------|-----------------|  
| `NewTincIPAMDriver` | receives a `context.Context`, a pointer to `TincNetworkState`, and a `TincNetworkConfig`. |  
| `GetCapabilities` | returns an `ipam.CapabilitiesResponse`; no external input beyond the logger. |  
| `GetDefaultAddressSpaces` | returns an `ipam.AddressSpacesResponse`; placeholder for future logic. |  
| `RequestPool` | consumes an `ipam.RequestPoolRequest`, uses `t.netByIPAMOptions(request.Options)` to fetch a network, and returns an `ipam.RequestPoolResponse`. |  
| `ReleasePool` | consumes an `ipam.ReleasePoolRequest`; currently only logs the request. |  
| `RequestAddress` | consumes an `ipam.RequestAddressRequest`, looks up a network by ID (`t.netByID(request.PoolID)`), calculates mask size, optionally handles gateway type requests, otherwise picks a random free IP via `getRandomIP`. |  
| `ReleaseAddress` | consumes an `ipam.ReleaseAddressRequest`; placeholder. |  
| `getRandomIP` / `randomIP` | helper functions that generate a random IP inside a subnet; used by `RequestAddress`. |  
  
---  
  
### TODO comments  
No explicit `TODO:` markers are present in the current file.  
  
---  
  
## Summary of major code parts  
  
### 1. Driver construction (`NewTincIPAMDriver`)  
Creates a new `TincIPAMDriver` instance, embedding the provided network state and initializing a Sugared logger with context `"tinc/ipam"`.  
  
### 2. Capabilities & address spaces  
* **GetCapabilities** – logs receipt of request and returns an `ipam.CapabilitiesResponse` indicating that no MAC address is required.  
* **GetDefaultAddressSpaces** – currently returns `nil`; intended to provide default address space information in the future.  
  
### 3. Pool handling  
* **RequestPool** – receives a pool request, fetches network data via `netByIPAMOptions`, and replies with the node ID, subnet string, and original options.  
* **ReleasePool** – logs release requests; implementation pending.  
  
### 4. Address allocation (`RequestAddress`)  
1. Looks up the target network by its pool ID using `t.netByID`.    
2. Computes the mask size of the subnet.    
3. If the request type is `"com.docker.network.gateway"`, it generates a gateway IP by incrementing the last octet (or 16th byte) and returns that address.    
4. Otherwise, it fetches occupied IPs with `n.OccupiedIPs(t.ctx)`, picks a random free IP via `getRandomIP`, and returns it.  
  
### 5. Address release (`ReleaseAddress`)  
Logs the request; actual logic to be added later.  
  
### 6. Random IP helpers  
* **getRandomIP** – iterates up to 1000 times, calling `randomIP` until an unused address is found.  
* **randomIP** – generates a random 32‑bit value within the subnet and converts it into an `IP4` struct.  
  
---  
  
All functions use the embedded logger for debugging and informational output. The driver currently supports basic pool request/response handling and simple IP allocation, with placeholders for future enhancements such as default address spaces and release logic.  
  
# insonmnia/worker/network/tinc_network.go  
**Package / Component**    
`network`  
  
---  
  
### Imports  
```go  
"bytes"  
"context"  
"errors"  
"fmt"  
"net"  
"strings"  
  
"github.com/docker/docker/api/types"  
"github.com/docker/docker/pkg/stdcopy"  
```  
  
These imports provide standard utilities (`bytes`, `context`, etc.) and Docker‑specific types for executing commands inside a container.  
  
---  
  
### External data / input sources    
| Variable | Description |  
|----------|-------------|  
| `t.NodeID` | Identifier of the tinc node (used in all commands) |  
| `t.ConfigPath` | Path to the tinc configuration file |  
| `t.TincContainerID` | Docker container ID where tinc runs |  
| `t.Pool.String()` | String representation of a subnet pool (used when starting tinc) |  
| `t.logger` | Logger used for debug / error messages |  
  
---  
  
### TODOs  
No explicit `TODO:` comments are present in the file, but potential improvements could be:  
- Validate that all command strings are correctly formed.  
- Add error handling for missing fields (`t.Pool`, etc.).  
- Consider refactoring repeated `runCommandWithOutput` calls into a helper.  
  
---  
  
## Summary of major code parts  
  
| Section | Purpose |  
|---------|---------|  
| **IP4 type & helpers** | Defines a lightweight IPv4 address representation and conversion to/from `net.IP`. |  
| **`newIP4(ip net.IP) IP4`** | Creates an `IP4` from a 4‑byte IPv4 address. |  
| **`(i *IP4) ToCommon()`** | Converts the internal byte fields back into a standard `net.IP`. |  
| **`(t *TincNetwork) Init(ctx)`** | Executes `tinc init …` to initialise the network inside the Docker container. |  
| **`Join(ctx)`** | Runs `tinc join …` to add an invitation for another node. |  
| **`Start(ctx, addr string)`** | Starts tinc with interface, subnet and log level options; uses `iface`, `addr` and pool data. |  
| **`Shutdown(ctx)`** | Kills the Docker container running tinc (`SIGKILL`). |  
| **`Stop(ctx)`** | Stops tinc inside the container via a batch command. |  
| **`Invite(ctx, inviteeID string)`** | Invites another node; returns the output of `tinc invite …`. |  
| **`OccupiedIPs(ctx)`** | Retrieves all currently occupied IPs by running start → dump → stop and parsing the output into a map of `IP4`. |  
| **`runCommand(ctx, name string, arg ...string)`** | Wrapper that calls `runCommandWithOutput` and discards its stdout/stderr. |  
| **`runCommandWithOutput(ctx, name string, arg ...string)`** | Executes an arbitrary command inside the Docker container, attaches to it, copies stdout/stderr via `stdcopy.StdCopy`, and returns both streams plus any error. |  
  
---  
  
The file implements a set of helper methods for managing a tinc network inside a Docker container, providing high‑level operations such as init, join, start/stop, invite, and IP discovery.  
  
# insonmnia/worker/network/tinc_state.go  
# Package `network`  
  
## Imports    
The file pulls in the following packages:  
  
| Import | Purpose |  
|--------|---------|  
| `context` | Context handling for async operations |  
| `encoding/json` | Marshal/unmarshal state data |  
| `errors` | Error creation and handling |  
| `fmt` | String formatting |  
| `net` | IP/CIDR parsing & manipulation |  
| `sync` | Read/write mutexes |  
| `github.com/docker/docker/api/types` | Docker container types |  
| `github.com/docker/docker/api/types/container` | Container configuration |  
| `github.com/docker/docker/api/types/network` | Network config |  
| `github.com/docker/docker/client` | Docker client API |  
| `github.com/docker/libkv` | Key‑value store abstraction |  
| `github.com/docker/libkv/store` | Store backend interface |  
| `github.com/docker/libkv/store/boltdb` | BoltDB backend registration |  
| `log "github.com/noxiouz/zapctx/ctxlog"` | Context‑aware logger (aliased as `log`) |  
| `github.com/sonm-io/core/insonmnia/structs` | Custom network spec struct |  
| `go.uber.org/zap` | Structured logging helpers |  
  
---  
  
## External Data / Input Sources    
* **Configuration** – `TincNetworkConfig` (passed to the constructor).    
* **State Path** – `config.StatePath` used by `makeStore`.    
* **Docker Client** – passed in as `client *client.Client`.    
* **Storage Backend** – BoltDB via `github.com/docker/libkv/store/boltdb`.    
  
---  
  
## TODOs    
No explicit `TODO:` comments are present. If future work is needed, add them here.  
  
---  
  
## Major Code Parts  
  
### 1. `TincNetworkState` struct  
```go  
type TincNetworkState struct {  
    ctx      context.Context  
    config   *TincNetworkConfig  
    mu       sync.RWMutex  
    cli      *client.Client  
    Networks map[string]*TincNetwork  
    Pools    map[string]*net.IPNet  
    logger   *zap.SugaredLogger  
    storage  store.Store  
}  
```  
* Holds the runtime state of all Tinc networks, a mutex for concurrency, and a BoltDB store.  
  
### 2. Helper – `defaultNet()`  
Provides a default CIDR block (`10.20.30.0/24`) used when no subnet is supplied.  
  
### 3. Constructor – `newTincNetworkState`  
Creates the state object, initializes the BoltDB store, loads existing data, and returns the ready instance.  
  
### 4. Network Creation – `InsertTincNetwork`  
* Parses a CIDR string from a `structs.NetworkSpec`.    
* Creates and starts a Docker container for Tinc.    
* Builds a `TincNetwork` object with all relevant fields (node ID, pool, invitation, etc.).    
* Stores the new network in the internal map.  
  
### 5. Lookup Helpers  
* `netByID(id string)` – fetches a network by its node ID.    
* `netByOptions(data map[string]interface{})` – extracts an ID from options and delegates to `netByID`.    
* `netByIPAMOptions(data map[string]string)` – similar but for IPAM‑style options.    
* `netByDockerID(id string)` – iterates over all networks to find one by its Docker container ID.  
  
### 6. Store Initialization – `makeStore`  
Registers BoltDB, creates a store backend with bucket `"sonm_tinc_driver_state"`, and returns it.  
  
### 7. Persistence – `load()` & `sync()`  
* `load` reads the stored state from key `"state"` into the struct, re‑injects client/logger references for each network, and handles errors.  
* `sync` marshals the current state to JSON and writes it back under key `"state"`.  
  
---  
  
All of these parts together allow a Tinc network driver to maintain its own state in BoltDB, create Docker containers on demand, and expose convenient lookup helpers.  
  
# insonmnia/worker/network/tinc_tuner.go  
**Package name:** `network`    
  
**Imports**  
  
```go  
import (  
	"context"  
	"errors"  
	"os"  
	"path/filepath"  
	"syscall"  
	"time"  
  
	"github.com/docker/docker/api/types"  
	"github.com/docker/docker/api/types/container"  
	"github.com/docker/docker/api/types/network"  
	"github.com/docker/docker/client"  
	"github.com/docker/go-connections/sockets"  
	"github.com/docker/go-plugins-helpers/ipam"  
	netdriver "github.com/docker/go-plugins-helpers/network"  
	log "github.com/noxiouz/zapctx/ctxlog"  
	"github.com/sonm-io/core/insonmnia/structs"  
	"go.uber.org/zap"  
)  
```  
  
**External data / input sources**  
  
| Source | Description |  
|--------|-------------|  
| `TincNetworkConfig` | Configuration for the Tinc network driver (used in `NewTinc`). |  
| `TincNetworkDriver`, `TincIPAMDriver` | Drivers that handle network and IPAM logic. |  
| `structs.NetworkSpec` | Network specification passed to `Tune`. |  
| `container.HostConfig` | Host configuration for Docker containers. |  
| `network.NetworkingConfig` | Docker networking config used when creating a network. |  
  
**TODO comments**  
  
No explicit TODO markers are present in the file.  
  
---  
  
## Summary of major code parts  
  
### 1. Type definitions  
- **`TincTuner`** – holds a Docker client, a Tinc network driver and an IPAM driver.  
- **`TincCleaner`** – used to clean up a created network; stores context, network ID and the same client.  
  
### 2. `NewTincTuner`  
Creates a new Docker environment client, initializes both drivers via `NewTinc`, builds a tuner instance and immediately starts its driver logic with `runDriver`.    
Returns a pointer to the initialized tuner or an error.  
  
### 3. `runDriver`  
- Creates directories for the network and IPAM plugin sockets.  
- Opens Unix sockets for each plugin using `sockets.NewUnixSocket`.  
- Builds handlers (`netdriver.NewHandler`, `ipam.NewHandler`) and starts goroutines that serve them until the context is cancelled.  
- Handles cleanup of listeners on context cancellation.  
  
### 4. `Tune`  
- Inserts a Tinc network into the driver with `InsertTincNetwork`.  
- Builds Docker options for both the main driver (`tinc`) and its IPAM plugin (`tincipam`).  
- Calls Docker’s `NetworkCreate` to create the network.  
- Registers endpoint settings in the provided `config.EndpointsConfig`.  
- Returns a `TincCleaner` that can later remove the created network.  
  
### 5. `GetCleaner`  
Retrieves an existing network mapping from the driver and returns a cleaner for it.  
  
### 6. `Tuned`  
Simple helper that checks whether a given network ID is present in the driver.  
  
### 7. `GenerateInvitation`  
Delegates to the underlying driver to generate an invitation string for a network ID.  
  
### 8. `Close` (on `TincCleaner`)  
Removes the network via Docker client, retrying up to ten times with exponential back‑off.  
  
### 9. `cloneOptions`  
Utility that copies a map of string keys/values – used internally by `Tune`.  
  
---  
  
This file implements the core logic for creating, serving and cleaning Tinc networks within the `network` package.  
  
# insonmnia/worker/network/tuner.go  
# Package / Component    
**network**  
  
## Imports  
```go  
import (  
	"context"  
  
	"github.com/docker/docker/api/types/container"  
	"github.com/docker/docker/api/types/network"  
	"github.com/sonm-io/core/insonmnia/structs"  
)  
```  
  
* `context` – standard Go context package for request-scoped values.    
* `github.com/docker/docker/api/types/container` – Docker container configuration types.    
* `github.com/docker/docker/api/types/network` – Docker networking configuration types.    
* `github.com/sonm-io/core/insonmnia/structs` – internal struct definitions, notably `NetworkSpec`.  
  
## External Data / Input Sources  
The file defines two interfaces that operate on:  
1. **`structs.NetworkSpec`** – a specification of the network to be tuned.  
2. **`container.HostConfig`** – Docker container host configuration.  
3. **`network.NetworkingConfig`** – Docker networking configuration.  
  
These types are passed into the `Tuner.Tune` method and used by other methods in the interface.  
  
## TODOs  
No explicit TODO comments were found in this file.  
  
---  
  
# Summary of Major Code Parts  
  
## Interface `Cleanup`  
```go  
type Cleanup interface {  
	Close() error  
}  
```  
*Purpose*: Represents a cleanup action that can be performed after tuning.    
*Method*: `Close()` – returns an error if the cleanup fails.  
  
## Interface `Tuner`  
```go  
type Tuner interface {  
	Tune(ctx context.Context, net *structs.NetworkSpec, hostConfig *container.HostConfig, netConfig *network.NetworkingConfig) (Cleanup, error)  
	GenerateInvitation(ID string) (*structs.NetworkSpec, error)  
	GetCleaner(ctx context.Context, ID string) (Cleanup, error)  
	Tuned(ID string) bool  
}  
```  
*Purpose*: Encapsulates all logic needed to prepare networking and bake options into Docker container and network configurations.    
*Methods*:  
- **`Tune`** – Main entry point; receives a context, a network spec, host config, and network config, then returns a `Cleanup` implementation and an error.  
- **`GenerateInvitation`** – Creates a new `NetworkSpec` based on an ID string (likely used to generate a unique invitation for the network).  
- **`GetCleaner`** – Retrieves a cleanup instance for a given ID; useful for cleaning up after tuning.  
- **`Tuned`** – Checks whether a particular ID has already been tuned.  
  
The file defines only interfaces, so implementations will be provided elsewhere in the package. The design suggests that `Tuner` is responsible for orchestrating Docker networking operations and that any implementation must provide cleanup logic via the `Cleanup` interface.  
  
