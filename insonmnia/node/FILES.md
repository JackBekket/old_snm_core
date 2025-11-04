# insonmnia/node/blacklist.go  
**Package / Component**    
`node`  
  
---  
  
### Imports  
| Package | Purpose |  
|---------|---------|  
| `context` | Provides the context type used in RPC calls. |  
| `errors` | Error handling for validation and return values. |  
| `fmt` | Formatting error messages. |  
| `github.com/ethereum/go-ethereum/crypto` | Ethereum crypto utilities (address conversion). |  
| `github.com/sonm-io/core/proto` | Core protocol types (`EthAddress`, `BlackListReply`, etc.). |  
| `github.com/sonm-io/core/util/xconcurrency` | Parallel execution helper. |  
  
---  
  
### External data / input sources  
* **remoteOptions** – holds remote services used by the API:  
  * `dwh` – a data‑warehouse client providing `GetBlacklist`.  
  * `eth` – an Ethereum service exposing a `Blacklist()` method.  
* **crypto.PubkeyToAddress** – converts a public key to an Ethereum address.  
* **xconcurrency.Run** – runs a function concurrently over a list of addresses.  
  
---  
  
### TODOs  
No explicit TODO comments are present in this file.    
(If future work is needed, add them here.)  
  
---  
  
## Summary of major code parts  
  
#### `newBlacklistAPI`  
Creates and returns a new instance of the API implementation.    
It simply stores the provided `remoteOptions` pointer for later use.  
  
```go  
func newBlacklistAPI(opts *remoteOptions) sonm.BlacklistServer {  
    return &blacklistAPI{ remotes: opts }  
}  
```  
  
#### `(m *blacklistAPI) List`  
Retrieves a blacklist reply from the remote data‑warehouse.    
* Validates that the supplied address is non‑zero.  
* Calls `GetBlacklist` on the dwh service with a request containing the user ID.  
  
```go  
func (m *blacklistAPI) List(ctx context.Context, addr *sonm.EthAddress) (*sonm.BlacklistReply, error) {  
    if addr.IsZero() { return nil, errors.New("address is empty") }  
    return m.remotes.dwh.GetBlacklist(ctx, &sonm.BlacklistRequest{UserID: addr})  
}  
```  
  
#### `(m *blacklistAPI) Remove`  
Removes a single address from the blacklist.    
* Calls the Ethereum service’s `Remove` method with the key and wrapped address.  
* Returns an empty response on success.  
  
```go  
func (m *blacklistAPI) Remove(ctx context.Context, addr *sonm.EthAddress) (*sonm.Empty, error) {  
    if err := m.remotes.eth.Blacklist().Remove(ctx, m.remotes.key, addr.Unwrap()); err != nil {  
        return nil, fmt.Errorf("failed to remove address from blacklist: %v", err)  
    }  
    return &sonm.Empty{}, nil  
}  
```  
  
#### `(m *blacklistAPI) Purge`  
Cleans the entire blacklist.    
* Computes the current node’s own Ethereum address.  
* Retrieves the full list via `List`.  
* Uses `xconcurrency.Run` to concurrently remove each listed address, collecting errors in a `ErrorByStringID` structure.  
  
```go  
func (m *blacklistAPI) Purge(ctx context.Context, req *sonm.Empty) (*sonm.ErrorByStringID, error) {  
    myAddr := crypto.PubkeyToAddress(m.remotes.key.PublicKey)  
    list, err := m.List(ctx, sonm.NewEthAddress(myAddr))  
    if err != nil { return nil, err }  
  
    status := sonm.NewTSErrorByStringID()  
    xconcurrency.Run(purgeConcurrency, list.GetAddresses(), func(elem interface{}) {  
        id := elem.(string)  
        addr, err := sonm.NewEthAddressFromHex(id)  
        if err != nil {  
            status.Append(id, fmt.Errorf("failed to parse eth address from string: %v", err))  
            return  
        }  
        _, err = m.Remove(ctx, addr)  
        status.Append(addr.Unwrap().Hex(), err)  
    })  
  
    return status.Unwrap(), nil  
}  
```  
  
---  
  
This file implements the core CRUD operations for a node‑level blacklist service.    
It will be combined with other node components to form the full package summary.  
  
# insonmnia/node/config.go  
**Package/Component:** `node`  
  
### Imports  
```go  
import (  
	"github.com/jinzhu/configor"          // YAML config loader  
	"github.com/sonm-io/core/accounts"  
	"github.com/sonm-io/core/blockchain"  
	"github.com/sonm-io/core/insonmnia/benchmarks"  
	"github.com/sonm-io/core/insonmnia/dwh"  
	"github.com/sonm-io/core/insonmnia/logging"  
	"github.com/sonm-io/core/insonmnia/matcher"  
	"github.com/sonm-io/core/insonmnia/npp"  
	"github.com/sonm-io/core/insonmnia/ssh"  
	"github.com/sonm-io/core/optimus"  
	"github.com/sonm-io/core/util/debug"  
)  
```  
  
### External Data / Input Sources  
* YAML configuration file – the `NewConfig` function loads a local node configuration from a given `.yaml` path using **configor**.  
  
### TODOs  
No explicit TODO comments are present in this snippet.    
(If future work is needed, add a section for TODO items.)  
  
---  
  
## Summary of Major Code Parts  
  
#### 1. Configuration Structures  
| Field | Type | YAML Key | Description |  
|-------|------|----------|-------------|  
| `HttpBindPort` | `uint16` | `http_bind_port` | HTTP server listening port (default: 15031) |  
| `BindPort` | `uint16` | `bind_port` | Node binding port (default: 15030) |  
| `AllowInsecureConnection` | `bool` | `allow_insecure_connection` | Flag for insecure connections |  
| `Node` | `nodeConfig` | `node` | Grouping of the above node-specific settings |  
| `NPP` | `npp.Config` | `npp` | Configuration for NPP component |  
| `Log` | `logging.Config` | `log` | Logging configuration |  
| `Blockchain` | `*blockchain.Config` | `blockchain` | Blockchain-related config (pointer) |  
| `Eth` | `accounts.EthConfig` | `ethereum` | Ethereum account settings (optional) |  
| `DWH` | `dwh.YAMLConfig` | `dwh` | Data warehouse YAML configuration |  
| `MetricsListenAddr` | `string` | `metrics_listen_addr` | Address for metrics listening (default: 127.0.0.1:14003) |  
| `Benchmarks` | `benchmarks.Config` | `benchmarks` | Benchmarking config |  
| `Matcher` | `*matcher.YAMLConfig` | `matcher` | Matcher component configuration (pointer) |  
| `Predictor` | `*optimus.PredictorConfig` | `predictor` | Optimus predictor settings (pointer) |  
| `Debug` | `*debug.Config` | `debug` | Debugging config (pointer) |  
| `SSH` | `*ssh.ProxyServerConfig` | `ssh` | SSH proxy server configuration (pointer) |  
  
#### 2. `NewConfig` Function  
- **Purpose:** Load a node configuration from a YAML file.  
- **Parameters:** `path string` – path to the YAML config file.  
- **Return Value:** `(*Config, error)` – pointer to the populated `Config` struct and any loading error.  
- **Implementation Details:**  
  - Instantiates an empty `Config`.  
  - Calls `configor.Load(cfg, path)` to parse the YAML into the struct.  
  - Returns the config or an error if parsing fails.  
  
---  
  
This file defines the core configuration structure for a node in the system and provides a helper function to load it from disk. The configuration is composed of several subcomponents (NPP, logging, blockchain, etc.) that are referenced by other parts of the package.  
  
# insonmnia/node/deals.go  
**Package / Component**    
`node.dealsAPI`  
  
---  
  
### Imports  
```go  
import (  
    "context"  
    "errors"  
    "fmt"  
    "time"  
  
    "github.com/ethereum/go-ethereum/crypto"  
    "github.com/sonm-io/core/insonmnia/auth"  
    "github.com/sonm-io/core/insonmnia/dwh"  
    "github.com/sonm-io/core/proto"  
    "github.com/sonm-io/core/util/xconcurrency"  
    "go.uber.org/zap"  
    "google.golang.org/grpc/codes"  
    "google.golang.org/grpc/status"  
)  
```  
The API relies on Ethereum crypto utilities, a local `auth` and `dwh` package for data‑warehouse access, protobuf definitions (`proto`) and concurrency helpers from `xconcurrency`. Logging is done with Uber’s Zap.  
  
---  
  
### External Data / Input Sources  
| Source | Purpose |  
|--------|---------|  
| `crypto.PubkeyToAddress` | Convert a public key to an Ethereum address. |  
| `dwh.GetDeals` | Retrieve deals by status, consumer or supplier. |  
| `eth.Market()` | Blockchain market operations: get deal info, close/open/quick‑buy deals, create change requests. |  
| `auth.ParseAddr` | Parse a worker address from a hex string. |  
| `xconcurrency.Run` | Parallel execution of closing multiple deals. |  
  
---  
  
### TODOs  
No explicit TODO comments were found in the file.  
  
---  
  
## Code Summary  
  
#### 1. `List`  
Fetches all deals where the current node is either supplier or consumer, merges them into a single reply and returns it.  
  
#### 2. `Status`  
Retrieves deal info from the blockchain; if the caller is the consumer, it also pulls extra data from the worker node via a temporary client.  
  
#### 3. `Finish` & `FinishDeals`  
Closes a single deal on the blockchain (`Finish`) or multiple deals in parallel (`FinishDeals`).    
The helper `finishDeals` uses `xconcurrency.Run` to close each deal concurrently and aggregates errors into an `ErrorByID`.  
  
#### 4. `PurgeDeals`  
Convenience wrapper that fetches all accepted deals for the current node, converts them into finish requests and delegates to `finishDeals`.  
  
#### 5. `Open`  
Opens a new deal by first retrieving ask order info, checking worker availability (unless forced), then calling the blockchain market’s `OpenDeal`.    
  
#### 6. `QuickBuy`  
Similar to `Open` but performs a quick‑buy operation: it calculates duration from the ask order or request, checks worker availability, executes `QuickBuy`, obtains supplier address, creates a worker client and finally fetches detailed deal info from that worker.  
  
#### 7. `ChangeRequestsList`  
Simple wrapper around `dwh.GetDealChangeRequests` to list change requests for a given deal ID.  
  
#### 8. `CreateChangeRequest`  
Creates a new change request on the blockchain: it ensures the caller is either consumer or master, fills missing duration/price fields, and returns the newly created request’s ID.  
  
#### 9. `ApproveChangeRequest`  
Approves an existing change request by creating a matching request with inverted order type (ASK ↔ BID).  
  
#### 10. `CancelChangeRequest`  
Cancels a change request via the blockchain market.  
  
#### 11. `invertOrderType`  
Utility that flips an `OrderType` value between ASK and BID.  
  
#### 12. `newDealsAPI`  
Constructor that returns a fully initialized `dealsAPI` instance, ready to serve as a `sonm.DealManagementServer`.  
  
---  
  
All functions together provide a complete CRUD interface for deals, change requests, and worker interactions within the node component of the Sonm‑NIA system.  
  
# insonmnia/node/market.go  
**Package / Component**    
`node`  
  
---  
  
### Imports  
| Package | Purpose |  
|---------|---------|  
| `context` | Go context handling for request/response lifecycles |  
| `errors` | Error creation and propagation |  
| `fmt` | String formatting for error messages |  
| `github.com/ethereum/go-ethereum/crypto` | Ethereum key/address utilities |  
| `github.com/sonm-io/core/insonmnia/dwh` | Data‑warehouse access (orders, etc.) |  
| `github.com/sonm-io/core/proto` | Protocol buffer definitions for orders and requests |  
| `github.com/sonm-io/core/util` | Utility helpers (BigInt parsing) |  
| `github.com/sonm-io/core/util/xconcurrency` | Parallel execution helper (`Run`) |  
| `go.uber.org/zap` | Structured logging |  
  
---  
  
### External Data / Input Sources  
* **Remote options** – `remoteOptions` struct provides:  
  * `dwh`: data‑warehouse client for order queries.  
  * `eth.Market()`: blockchain market interface (order placement, cancellation).  
  * `key.PublicKey`: Ethereum public key used to identify the author of orders.  
* **Protocol buffers** – request/response types from `github.com/sonm-io/core/proto`:  
  * `Count`, `GetOrdersReply`, `ID`, `BidOrder`, `Empty`, `OrderIDs`.  
* **Benchmark list** – accessed via `remotes.benchList.MapByCode()` for mapping benchmark codes to values.  
  
---  
  
### TODOs  
No explicit `TODO:` comments were found in the file.    
(If future work is needed, add a section here.)  
  
---  
  
## Summary of Major Code Parts  
  
#### 1. **`marketAPI` struct & constructor**  
```go  
type marketAPI struct {  
	remotes       *remoteOptions  
	workerCreator workerClientCreator  
	log           *zap.SugaredLogger  
}  
```  
* Holds references to remote options, a worker creator, and a logger.  
* `newMarketAPI(opts *remoteOptions) sonm.MarketServer` returns an initialized instance.  
  
#### 2. **GetOrders**  
```go  
func (m *marketAPI) GetOrders(ctx context.Context, req *sonm.Count) (*sonm.GetOrdersReply, error)  
```  
* Builds a filter for active BID orders by the current author.  
* Calls `dwh.GetOrders` to fetch data from the warehouse.  
* Aggregates results into a `GetOrdersReply`.  
  
#### 3. **GetOrderByID**  
```go  
func (m *marketAPI) GetOrderByID(ctx context.Context, req *sonm.ID) (*sonm.Order, error)  
```  
* Parses an ID string to a BigInt and retrieves the order from the blockchain market.  
  
#### 4. **CreateOrder**  
```go  
func (m *marketAPI) CreateOrder(ctx context.Context, req *sonm.BidOrder) (*sonm.Order, error)  
```  
* Maps benchmark codes to values, builds a `Benchmarks` struct.  
* Constructs an `Order` with all required fields (type, status, author, counterparty, duration, price, netflags, identity level, blacklist, tag).  
* Places the order on the blockchain via `eth.Market().PlaceOrder`.  
* Starts a goroutine that creates a deal for the new order and logs success.  
  
#### 5. **CancelOrder**  
```go  
func (m *marketAPI) CancelOrder(ctx context.Context, req *sonm.ID) (*sonm.Empty, error)  
```  
* Cancels an individual order by ID on the blockchain.  
* Returns an empty response upon success.  
  
#### 6. **Purge & PurgeVerbose**  
```go  
func (m *marketAPI) Purge(ctx context.Context, req *sonm.Empty) (*sonm.Empty, error)  
func (m *marketAPI) PurgeVerbose(ctx context.Context, _ *sonm.Empty) (*sonm.ErrorByID, error)  
```  
* `Purge` is a thin wrapper that calls `PurgeVerbose`.  
* `PurgeVerbose` fetches all active orders and delegates to `cancelOrders`.  
  
#### 7. **CancelOrders & cancelOrders**  
```go  
func (m *marketAPI) CancelOrders(ctx context.Context, req *sonm.OrderIDs) (*sonm.ErrorByID, error)  
func (m *marketAPI) cancelOrders(ctx context.Context, ids []*sonm.BigInt) (*sonm.ErrorByID, error)  
```  
* `CancelOrders` accepts a list of order IDs and forwards them to the internal helper.  
* `cancelOrders` performs concurrent cancellations using `xconcurrency.Run`, with a concurrency level defined by `purgeConcurrency`.  
  
---  
  
**End of summary.**  
  
# insonmnia/node/master.go  
## Package / Component    
**node**  
  
### Imports  
```go  
import (  
	"context"  
	"fmt"  
  
	"github.com/sonm-io/core/proto"  
	"go.uber.org/zap"  
)  
```  
  
### External Data, Input Sources  
| Source | Description |  
|--------|-------------|  
| `remoteOptions` | Holds configuration for remote services (DWH, Eth, etc.) and a logger. |  
| `zap.SugaredLogger` | Logging facility used throughout the API. |  
| `sonm.EthAddress`, `sonm.WorkerListReply`, `sonm.Empty`, `sonm.WorkerRemoveRequest` | Protobuf types from the `github.com/sonm-io/core/proto` package. |  
  
### TODOs  
1. **Top of file** – *DWH is required to implement this service* (TODO(sshaman1101)).  
2. Inside `WorkersList` – *pagination*.  
  
---  
  
## Summary of Major Code Parts  
  
#### 1. `masterMgmtAPI` struct    
Defines the API implementation for master‑management operations. It stores:  
- `remotes`: a pointer to `remoteOptions`, providing access to DWH and Eth services.  
- `log`: a SugaredLogger instance for logging.  
  
#### 2. `newMasterManagementAPI` constructor    
Creates a new `masterMgmtAPI` instance, wiring the provided `remoteOptions` into the struct and returning it as a `sonm.MasterManagementServer`.  
  
#### 3. `WorkersList` method    
- Calls DWH (`GetWorkers`) to fetch workers for a given master address.  
- Wraps the result in a `WorkerListReply` and returns it, with an error path that logs failures.  
  
#### 4. `WorkerConfirm` method    
- Confirms a worker on the blockchain via Eth’s Market service.  
- Returns an empty response or an error if the call fails.  
  
#### 5. `WorkerRemove` method    
- Removes a worker from the blockchain using Eth’s Market service.  
- Similar to confirm, it returns an empty response or an error.  
  
---  
  
All methods are straightforward wrappers around remote services, providing a clean API for master‑management operations within the `node` package.  
  
# insonmnia/node/mod.go  
**Package / Component Name**    
`node`  
  
---  
  
### Imports  
| Package | Purpose |  
|---------|---------|  
| `context` | Provides context handling for long‑lived operations (e.g., certificate rotation). |  
| `crypto/ecdsa` | Handles ECDSA keys used for TLS credentials. |  
| `crypto/tls` | TLS configuration and transport credentials. |  
| `fmt` | Standard formatting utilities. |  
| `github.com/sonm-io/core/util` | Core utilities (e.g., certificate rotator, TLS helper). |  
| `github.com/sonm-io/core/util/debug` | Debugging helpers for pprof. |  
| `github.com/sonm-io/core/util/xgrpc` | gRPC transport credentials wrapper. |  
| `golang.org/x/sync/errgroup` | Concurrency helper for running multiple goroutines. |  
| `google.golang.org/grpc/credentials` | Transport credentials interface used by gRPC. |  
  
---  
  
### External Data / Input Sources  
* **Config** – configuration struct that contains node settings, Ethereum key loader (`cfg.Eth.LoadKey()`), and SSH options.  
* **Option** – functional option type for configuring the Node (used in `newOptions()`).    
* **ServerOption** – functional option type used when creating a server instance.    
* **RemoteOptions** – struct returned by `newRemoteOptions` that bundles gRPC transport credentials, logging, and market data.    
* **Services** – collection of services built by `newServices`.    
  
---  
  
### TODOs  
No explicit `TODO:` comments were found in the provided snippet.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. Node struct  
```go  
type Node struct {  
    cfg    *Config  
    server *Server  
}  
```  
* Holds a reference to the node configuration and its underlying gRPC/REST server instance.  
  
### 2. `New` – Node constructor  
* **Purpose**: Build a fully configured `Node` ready for serving.  
* **Key steps**:  
  1. Apply functional options (`newOptions()`).  
  2. Load an ECDSA key from the configuration (`cfg.Eth.LoadKey()`).  
  3. Create TLS credentials and config via `newTLSWithConfig`.  
  4. Build remote gRPC options with `newRemoteOptions`, passing in transport credentials, logging, etc.  
  5. Assemble a list of server options (gRPC, REST, metrics, logging, optional QUIC/secure modes).  
  6. Conditionally add SSH support if SSH config is present.  
  7. Instantiate the underlying server with `newServer`.  
  8. Return the constructed `Node` instance.  
  
### 3. `Serve` – Node runtime  
* **Purpose**: Start serving both debug pprof and the main server concurrently.  
* Uses an `errgroup.WithContext` to run two goroutines:  
  - Debug pprof via `debug.ServePProf`.  
  - Main server via `m.server.Serve`.  
* Waits for all goroutines to finish before returning.  
  
### 4. TLS helpers  
```go  
func newTLS(ctx context.Context, privateKey *ecdsa.PrivateKey) (credentials.TransportCredentials, error)  
func newTLSWithConfig(ctx context.Context, privateKey *ecdsa.PrivateKey) (credentials.TransportCredentials, *tls.Config, error)  
```  
* `newTLS` creates a transport credential from a key and returns it.  
* `newTLSWithConfig` does the same but also returns the underlying TLS config for further use (e.g., in SSH options).  
  
---  
  
**<end_of_output>**  
  
# insonmnia/node/monitoring.go  
**Package & Imports**    
- **Package name:** `node`    
- **Imports:**  
  ```go  
  import (  
      "context"  
  
      "github.com/sonm-io/core/insonmnia/npp"  
      "github.com/sonm-io/core/proto"  
  )  
  ```  
  
---  
  
### External Data / Input Sources    
| Source | Description |  
|--------|-------------|  
| `nppDialer` | A pointer to an `npp.Dialer`, used to fetch NPP metrics. |  
| `*sonm.Empty` | Empty request payload for the RPC call (no data needed). |  
  
---  
  
### TODOs    
No explicit TODO comments were found in this file.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. Service struct  
```go  
type monitoringService struct {  
    nppDialer *npp.Dialer  
}  
```  
* Holds a reference to an NPP dialer, which is responsible for communicating with the underlying NPP system.*  
  
### 2. Constructor `newMonitoringAPI`  
```go  
func newMonitoringAPI(opts *remoteOptions) *monitoringService {  
    return &monitoringService{  
        nppDialer: opts.nppDialer,  
    }  
}  
```  
* Creates a new monitoring service instance, wiring the dialer from the provided options.*  
  
### 3. RPC method `MetricsNPP`  
```go  
func (m *monitoringService) MetricsNPP(context.Context, *sonm.Empty) (*sonm.NPPMetricsReply, error) {  
    metrics, err := m.nppDialer.Metrics()  
    if err != nil {  
        return nil, err  
    }  
  
    metricsResponse := map[string]*sonm.NamedMetrics{}  
  
    for addr, metric := range metrics {  
        addrMetrics := make([]*sonm.NamedMetric, 0)  
        for _, addrMetric := range metric {  
            addrMetrics = append(addrMetrics, &sonm.NamedMetric{  
                Name:   addrMetric.Name,  
                Metric: addrMetric.Metric,  
            })  
        }  
  
        metricsResponse[addr] = &sonm.NamedMetrics{Metrics: addrMetrics}  
    }  
  
    return &sonm.NPPMetricsReply{Metrics: metricsResponse}, nil  
}  
```  
* Calls the dialer to obtain raw NPP metrics, then transforms them into a map keyed by address.    
  * Each address maps to a slice of `NamedMetric` objects (name + metric value).    
  * The final reply is wrapped in a `sonm.NPPMetricsReply`.    
  
---  
  
**<end_of_output>**  
  
# insonmnia/node/options.go  
## Package / Component    
**node**  
  
### Imports    
| Import | Purpose |  
|--------|---------|  
| `crypto/ecdsa` | ECDSA key handling for authentication |  
| `crypto/sha256` | SHA‑256 hashing used in REST secure option |  
| `crypto/tls` | TLS configuration for QUIC support |  
| `github.com/ethereum/go-ethereum/crypto` | Ethereum crypto utilities (e.g., PubkeyToAddress) |  
| `github.com/sonm-io/core/blockchain` | Blockchain market API reference |  
| `github.com/sonm-io/core/insonmnia/auth` | Authenticator for gRPC server |  
| `github.com/sonm-io/core/insonmnia/ssh` | SSH proxy server implementation |  
| `github.com/sonm-io/core/util/rest` | REST client/server utilities |  
| `github.com/sonm-io/core/util/xgrpc` | gRPC extensions and options |  
| `go.uber.org/zap` | Structured logging (zap) |  
| `google.golang.org/grpc/credentials` | Credentials for secure gRPC |  
  
---  
  
## External Data / Input Sources    
* `ssh.ProxyServerConfig` – configuration passed to the SSH proxy server.    
* `blockchain.MarketAPI` – market API used by the SSH proxy.    
* `xgrpc.TransportCredentials` – credentials for gRPC authentication.    
  
---  
  
## TODOs    
No explicit TODO comments were found in this file.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. Options & Constructor  
```go  
type options struct { log *zap.Logger }  
func newOptions() *options { ... }  
```  
* Holds a logger used by the node component.  
* `newOptions` creates a default instance with a no‑op zap logger.  
  
### 2. WithLog Option  
```go  
func WithLog(log *zap.Logger) Option { ... }  
```  
* Returns an `Option` closure that sets the logger in an `options` value.  
  
---  
  
### 3. Server Options & Constructor  
```go  
type serverOptions struct {  
    allowGRPC         bool  
    optionsGRPC       []xgrpc.ServerOption  
    allowREST         bool  
    optionsREST       []rest.Option  
    exposeGRPCMetrics bool  
    EnableQUIC        bool  
    TLSConfig         *tls.Config  
    sshProxy          SSHServer  
    log               *zap.Logger  
}  
func newServerOptions() *serverOptions { ... }  
```  
* Encapsulates all configuration flags and slices for gRPC, REST, QUIC, and SSH.  
* `newServerOptions` initializes defaults: a nil SSH proxy and nop logger.  
  
---  
  
### 4. WithGRPCServer Option  
```go  
func WithGRPCServer() ServerOption { ... }  
```  
* Enables gRPC support by setting `allowGRPC = true`.  
  
### 5. WithGRPCSecure Option  
```go  
func WithGRPCSecure(credentials credentials.TransportCredentials, key *ecdsa.PrivateKey) ServerOption { ... }  
```  
* Adds a secure credential to the gRPC options slice using an Ethereum wallet authenticator.  
* Uses the provided ECDSA key to derive an address via `crypto.PubkeyToAddress`.  
  
### 6. WithQUIC Option  
```go  
func WithQUIC(cfg *tls.Config) ServerOption { ... }  
```  
* Enables QUIC support and stores a TLS configuration.  
  
### 7. WithRESTServer Option  
```go  
func WithRESTServer() ServerOption { ... }  
```  
* Sets the flag to enable REST server functionality.  
  
### 8. WithRESTSecure Option  
```go  
func WithRESTSecure(key *ecdsa.PrivateKey) ServerOption { ... }  
```  
* Creates an AES encoder/decoder from a SHA‑256 hash of the ECDSA key.  
* Appends the resulting rest options (encoder & decoder) to `optionsREST`.  
  
### 9. WithGRPCServerMetrics Option  
```go  
func WithGRPCServerMetrics() ServerOption { ... }  
```  
* Enables exposure of gRPC metrics.  
  
### 10. WithSSH Option  
```go  
func WithSSH(cfg ssh.ProxyServerConfig, privateKey *ecdsa.PrivateKey,  
    credentials *xgrpc.TransportCredentials, market blockchain.MarketAPI,  
    log *zap.SugaredLogger) ServerOption { ... }  
```  
* Instantiates an SSH proxy server via `ssh.NewSSHProxyServer`.  
* Stores the resulting server in `sshProxy` and sets the logger.  
  
### 11. WithServerLog Option  
```go  
func WithServerLog(log *zap.Logger) ServerOption { ... }  
```  
* Sets a custom logger for the server options struct.  
  
---  
  
All option functions return a `ServerOption`, which is a function that mutates a `serverOptions` instance and may return an error.    
These closures allow flexible, composable configuration of the node component.  
  
# insonmnia/node/profiles.go  
## Package / Component    
**node**  
  
### Imports  
```go  
import (  
	"context"  
  
	"github.com/sonm-io/core/proto"  
)  
```  
* `context` – standard Go context handling.  
* `github.com/sonm-io/core/proto` – contains the protobuf definitions and client interfaces used by this component.  
  
---  
  
## External Data / Input Sources    
| Function | Description |  
|----------|-------------|  
| `newProfileAPI` | Creates a new instance of `profileAPI`, wiring it with remote options. |  
| `List` | Calls the data‑warehouse (`dwh`) to fetch a list of profiles based on a request. |  
| `Status` | Retrieves detailed profile information for a given Ethereum address. |  
| `RemoveAttribute` | Removes a certificate from the profile registry using an ID and returns an empty response. |  
  
---  
  
## TODOs    
No explicit TODO comments are present in this file.  
  
---  
  
## Summary of Major Code Parts    
  
### 1. `profileAPI` struct    
* Holds a pointer to `remoteOptions`, which presumably contains configuration for remote services (e.g., data‑warehouse, Ethereum registry).*  
  
### 2. Constructor – `newProfileAPI`    
* Accepts a `*remoteOptions` argument and returns an implementation of the `sonm.ProfilesServer` interface.    
* Simply stores the options in the struct and hands it back to callers.*  
  
### 3. List Method – `List(ctx, req)`    
* Delegates to `p.remotes.dwh.GetProfiles`, passing along the context and request.    
* Returns a `*sonm.ProfilesReply` and an error (if any).*  
  
### 4. Status Method – `Status(ctx, addr)`    
* Calls `GetProfileInfo` on the data‑warehouse component of `remotes`.    
* Provides profile details for a given Ethereum address.*  
  
### 5. RemoveAttribute Method – `RemoveAttribute(ctx, id)`    
* Invokes `ProfileRegistry().RemoveCertificate` to delete a certificate identified by a big integer ID.    
* Returns an empty response wrapper (`*sonm.Empty`) and any error that occurs.*  
  
---  
  
All functions together provide CRUD‑style operations for profile data within the node package, acting as a thin wrapper around remote services defined in `remoteOptions`.  
  
# insonmnia/node/remote.go  
# Package / Component    
**node**  
  
## Imports  
```go  
import (  
	"context"  
	"crypto/ecdsa"  
	"fmt"  
	"io"  
  
	"github.com/ethereum/go-ethereum/common"  
	"github.com/sonm-io/core/blockchain"  
	"github.com/sonm-io/core/insonmnia/auth"  
	"github.com/sonm-io/core/insonmnia/benchmarks"  
	"github.com/sonm-io/core/insonmnia/matcher"  
	"github.com/sonm-io/core/insonmnia/npp"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util"  
	"github.com/sonm-io/core/util/xgrpc"  
	"go.uber.org/zap"  
)  
```  
  
## External Data / Input Sources    
| Field | Type | Description |  
|-------|------|-------------|  
| `cfg` | *Config | Configuration for the node, including NPP and matcher settings |  
| `key` | *ecdsa.PrivateKey | ECDSA key used for authentication & signing |  
| `eth` | blockchain.API | Ethereum API client (market operations) |  
| `dwh` | sonm.DWHClient | DWH (Distributed Worker Hub?) client |  
| `nppDialer` | *npp.Dialer | NPP dialer to connect to workers |  
| `workerCreator` | workerClientCreator | Factory function that creates a `workerClient` from an address |  
| `benchList` | benchmarks.BenchList | List of benchmark data for workers |  
| `orderMatcher` | matcher.Matcher | Matcher used to find orders / deals |  
| `log` | *zap.SugaredLogger | Logger instance |  
  
## TODOs    
No explicit `TODO:` comments were found in the file.  
  
---  
  
# Summary of Major Code Parts  
  
### 1. `workerClient` struct  
A composite client that bundles three sonm clients:  
- `sonm.WorkerClient`  
- `sonm.WorkerManagementClient`  
- `sonm.InspectClient`  
  
This struct is used throughout to interact with a worker node.  
  
### 2. `remoteOptions` struct    
Describes all options needed for remote operations, including configuration, key, blockchain API, DWH client, NPP dialer, factory function, benchmark list, matcher and logger.  
  
### 3. `getWorkerClientForDeal`  
*Purpose*: Given a deal ID (string), it:  
1. Parses the ID into a big integer.  
2. Retrieves deal info from the Ethereum market via `eth.Market().GetDealInfo`.  
3. Checks that the deal is not closed.  
4. Calls `getWorkerClientByEthAddr` to obtain a worker client for the supplier address found in the deal.  
5. Returns the client, its closer and any error.  
  
### 4. `getWorkerClientByEthAddr`  
*Purpose*: Creates a `workerClient` by dialing an NPP address:  
- Uses the factory closure stored in `remoteOptions.workerCreator`.  
- The closure itself dials via `nppDialer`, authenticates with ECDSA key, creates an xgrpc client and wraps it into a `workerClient`.  
  
### 5. `isWorkerAvailable`  
*Purpose*: Checks whether a worker at a given Ethereum address is reachable:  
1. Calls `getWorkerClientByEthAddr`.  
2. Defer-closes the returned closer.  
3. Invokes `.Status` on the worker client and returns true if no error.  
  
### 6. `newRemoteOptions`  
*Purpose*: Builds a fully‑initialized `remoteOptions` instance:  
- Creates an NPP dialer with rendezvous, relay, and logger options.  
- Defines a factory closure (`workerFactory`) that dials to a worker address and builds the composite client.  
- Instantiates a DWH client via xgrpc.  
- Creates an Ethereum API client with market support.  
- Builds a benchmark list from configuration.  
- Optionally creates a matcher if `cfg.Matcher` is provided; otherwise uses a disabled matcher.  
- Returns the populated struct.  
  
This function is the entry point for setting up all remote communication needed by the node component.  
  
# insonmnia/node/server.go  
# Package `node`  
  
**Imports**  
  
```go  
"context"  
"crypto/tls"  
"fmt"  
"net"  
"strings"  
"sync"  
  
"github.com/grpc-ecosystem/go-grpc-prometheus"  
"github.com/lucas-clemente/quic-go"  
"github.com/sonm-io/core/util/defergroup"  
"github.com/sonm-io/core/util/rest"  
"github.com/sonm-io/core/util/xgrpc"  
"github.com/sonm-io/core/util/xnet"  
"go.uber.org/zap"  
"golang.org/x/sync/errgroup"  
"google.golang.org/grpc"  
```  
  
**External data / input sources**  
  
| Symbol | Source | Notes |  
|--------|---------|-------|  
| `nodeConfig` | defined in another file of the same package | holds configuration for ports and TLS options |  
| `ServerOption` | constructor options passed to `newServer` | contains flags, TLS config, logging, etc. |  
| `SSHServer` | interface implemented by a concrete SSH server | used for SSH handling inside `Serve()` |  
  
**TODO comments**  
  
No explicit TODO markers were found in this file.  
  
---  
  
## Types and structs  
  
### `LocalEndpoints`  
```go  
type LocalEndpoints struct {  
	GRPC []net.Addr  
	QRPC []net.Addr  
	REST []net.Addr  
}  
```  
Holds the local addresses for gRPC, QUIC‑gRPC and REST listeners.  
  
### `serverNetwork`  
```go  
type serverNetwork struct {  
	mu            sync.Mutex  
	ListenersGRPC []net.Listener  
	ListenersQRPC []net.PacketConn  
	ListenersREST []net.Listener  
}  
```  
Encapsulates the raw network listeners.    
*`newServerNetwork`* builds a new instance from slices of listeners.    
*`LocalEndpoints()`* converts the listener slices into `LocalEndpoints`.    
*`localQRPCAddrs()`* extracts addresses from QUIC packet connections.    
*`Pop()`* returns the current network and clears its internal slices.  
  
### `Services`  
```go  
type Services interface {  
	RegisterGRPC(server *grpc.Server) error  
	RegisterREST(server *rest.Server) error  
	Interceptor() grpc.UnaryServerInterceptor  
	StreamInterceptor() grpc.StreamServerInterceptor  
	Run(ctx context.Context) error  
}  
```  
Defines the API that a node must expose: gRPC, REST and SSH handling.  
  
### `SSHServer`  
```go  
type SSHServer interface {  
	Serve(ctx context.Context) error  
}  
```  
A minimal interface for an SSH server used by `Server`.  
  
### `Server`  
```go  
type Server struct {  
	network   *serverNetwork  
	endpoints LocalEndpoints  
  
	services Services  
	serverGRPC *grpc.Server  
	serverREST *rest.Server  
	serverSSH  SSHServer  
  
	tlsConfig *tls.Config  
  
	log *zap.SugaredLogger  
}  
```  
Main server type that owns the network, endpoints, gRPC/REST servers and an SSH proxy.    
It also keeps a TLS configuration for QUIC handling.  
  
---  
  
## Constructor `newServer`  
  
```go  
func newServer(cfg nodeConfig, services Services, options ...ServerOption) (*Server, error)  
```  
  
* Builds a list of listeners:  
  * TCP loopback on `cfg.BindPort` → gRPC  
  * UDP packet loopback on same port → QUIC‑gRPC  
  * TCP loopback on `cfg.HttpBindPort` → REST  
* Wraps them into a `serverNetwork`.  
* Creates the `Server` struct, wiring in the provided services and options.  
* If `opts.allowGRPC`, registers gRPC server with several interceptors (trace, request log, verify, unary/stream).  
* If `opts.allowREST`, registers REST server similarly.  
* Optionally registers a Prometheus metrics collector for gRPC.  
* Returns the fully configured server.  
  
---  
  
## Methods of `Server`  
  
### `LocalEndpoints()`  
```go  
func (m *Server) LocalEndpoints() LocalEndpoints  
```  
Simply returns the stored endpoints.  
  
### `Serve(ctx context.Context)`  
```go  
func (m *Server) Serve(ctx context.Context) error  
```  
* Pops the current network from the server.  
* Starts four goroutines:  
  1. gRPC listeners (`serveGRPC`)  
  2. QUIC‑gRPC listeners (`serveQUIC`)  
  3. REST listeners (`serveHTTP`)  
  4. SSH proxy (`m.services.Run(ctx)`) and `m.serverSSH.Serve(ctx)`  
* Waits for context cancellation, then closes the server.  
  
### `serveGRPC`  
```go  
func (m *Server) serveGRPC(ctx context.Context, listeners ...net.Listener) error  
```  
Iterates over gRPC listeners, logs each address, serves the gRPC server on them, and returns when all are finished.  
  
### `serveQUIC`  
```go  
func (m *Server) serveQUIC(ctx context.Context, conns ...net.PacketConn) error  
```  
Creates a QUIC listener for each packet connection, logs it, and serves the same gRPC server over QUIC. Uses `quic.Listen` from the external library.  
  
### `serveHTTP`  
```go  
func (m *Server) serveHTTP(ctx context.Context, listeners ...net.Listener) error  
```  
Starts serving REST on all provided listeners; a goroutine closes them when the context is done.  
  
### `close()`  
```go  
func (m *Server) close()  
```  
Stops gRPC and REST servers if they exist.  
  
---  
  
## Helper functions  
  
| Function | Purpose |  
|----------|---------|  
| `toLocalAddrs(listeners []net.Listener)` | Convert a slice of listeners to a slice of addresses. |  
| `formatListeners(listeners []net.Listener)` | Produce a human‑readable string for logging. |  
| `closeListeners(listeners []net.Listener)` | Close all TCP listeners. |  
| `closePacketConns(connections []net.PacketConn)` | Close all UDP packet connections. |  
  
These helpers are used throughout the file to keep code tidy.  
  
---  
  
The file implements the core networking layer of a *LocalNode* instance: it creates loopback listeners for gRPC, QUIC‑gRPC and REST, exposes them through a `Server` struct, and provides methods to start/stop all services. The design keeps network state in a dedicated `serverNetwork`, allows easy extension via the `Services` interface, and logs each step with Zap.  
  
# insonmnia/node/server_test.go  
**Package / Component**    
`node`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"crypto/ecdsa"  
	"testing"  
  
	"github.com/ethereum/go-ethereum/crypto"  
	"github.com/golang/mock/gomock"  
	"github.com/sonm-io/core/insonmnia/auth"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util"  
	"github.com/sonm-io/core/util/xgrpc"  
	"github.com/stretchr/testify/assert"  
	"github.com/stretchr/testify/require"  
	"google.golang.org/grpc"  
	"google.golang.org/grpc/codes"  
	"google.golang.org/grpc/credentials"  
	"google.golang.org/grpc/status"  
)  
```  
  
* **context** – standard Go context handling.    
* **crypto/ecdsa** – ECDSA key generation and usage.    
* **testing** – Go testing framework.    
* **github.com/ethereum/go-ethereum/crypto** – Ethereum crypto helpers (key generation, address conversion).    
* **github.com/golang/mock/gomock** – Mocking utilities for tests.    
* **github.com/sonm-io/core/insonmnia/auth** – Wallet authenticator used in gRPC client.    
* **github.com/sonm-io/core/proto** – Protobuf definitions (used via `sonm` package).    
* **github.com/sonm-io/core/util** – Utility helpers for certificates and TLS.    
* **github.com/sonm-io/core/util/xgrpc** – gRPC client helper.    
* **github.com/stretchr/testify/assert & require** – Assertions in tests.    
* **google.golang.org/grpc, codes, credentials, status** – gRPC core types.  
  
---  
  
### External data / input sources  
| Source | Purpose |  
|--------|---------|  
| `util.NewHitlessCertRotator` | Generates a TLS certificate from an ECDSA key. |  
| `util.NewTLS` | Wraps the cert into a `credentials.TransportCredentials`. |  
| `sonm.NewMockMarketServer` | Mock gRPC server for market service. |  
| `NewMockServices` | Creates mock services container used by `newServer`. |  
| `WithGRPCServer()` & `WithGRPCSecure(...)` | Server options passed to `newServer`. |  
| `xgrpc.NewClient` | Client helper that connects to a gRPC endpoint. |  
| `sonm.NewMarketClient(conn)` | Creates a market client from the connection. |  
  
---  
  
### TODOs  
No explicit `TODO:` comments were found in this file.  
  
---  
  
## Summary of major code parts  
  
### 1. Interceptor helpers    
*`nopInterceptor`* – Simple unary interceptor that forwards the request to the handler.    
*`nopStreamInterceptor`* – Stream interceptor that simply calls the stream handler.  
  
These are used as mock interceptors for gRPC server registration in `servicesMock`.  
  
---  
  
### 2. TLS & key helpers    
*`newTestTLS(t *testing.T, privateKey *ecdsa.PrivateKey)`*    
- Calls `util.NewHitlessCertRotator` to create a cert rotator and then wraps it with `util.NewTLS`.    
- Returns the credentials used for secure gRPC connections.  
  
*`newTestKey(t *testing.T) *ecdsa.PrivateKey`*    
- Generates an ECDSA key via `crypto.GenerateKey()` and returns it.    
  
Both helpers are used throughout the tests to create a fresh key/certificate pair per test case.  
  
---  
  
### 3. Mock services setup – `servicesMock(c *gomock.Controller)`    
Creates a mock market server (`marketServer`) that expects `GetOrderByID` calls any number of times and returns an empty order.    
Registers this mock with the gRPC server via:  
```go  
services.EXPECT().RegisterGRPC(gomock.Any()).Times(1).Return(nil).Do(func(server *grpc.Server) error {  
	sonm.RegisterMarketServer(server, marketServer)  
	return nil  
})  
```  
Also registers a stream interceptor and a REST placeholder (currently unused).    
Returns the fully configured mock services instance.  
  
---  
  
### 4. Test cases  
  
| Test | Purpose | Key actions |  
|------|---------|-------------|  
| **TestConnectWithoutTLS** | Verify that a server can be started without TLS credentials, then connect via plain gRPC client and receive an error (expected). | - Create mock services.<br>- Start server with `WithGRPCServer()` only.<br>- Serve in goroutine.<br>- Connect using `xgrpc.NewClient` with no transport credentials.<br>- Call `GetOrderByID`; expect error. |  
| **TestConnectWithValidKeyWithoutWallet** | Same as above but the client uses a valid TLS key (no wallet authenticator). | - Use `newTestTLS(t, key)` for server.<br>- Connect with same TLS credentials.<br>- Expect successful call returning empty order. |  
| **TestConnectWithInvalidKeyWithoutWallet** | Test connection failure when client uses an *invalid* key (different from server’s key). | - Server uses one key; client uses a different key.<br>- Connect and expect error again. |  
| **TestConnectWithInvalidKeyWithWallet** | Same as previous but the client authenticates with a wallet authenticator created from its own key. | - Client creates `authenticator` via `auth.NewWalletAuthenticator`.<br>- Connect using that authenticator.<br>- Expect error (client’s key not matching server). |  
| **TestConnectWithValidKeyWithWallet** | Final test: client uses the *same* key as the server and authenticates with a wallet authenticator. | - Client re‑uses the same key used by the server.<br>- Connect via authenticator.<br>- Expect successful call returning empty order. |  
  
All tests follow the same pattern:  
1. Create mock services.  
2. Start a new server (`newServer(nodeConfig{}, ...)`).  
3. Serve in background goroutine.  
4. Use `xgrpc.NewClient` to connect to the first gRPC endpoint returned by `server.LocalEndpoints()`.  
5. Call `sonm.NewMarketClient(conn).GetOrderByID(...)`.  
6. Assert on error, result and status code.  
  
The tests validate that:  
* The server correctly registers the market service.  
* TLS credentials are properly applied.  
* Wallet authentication works when provided.  
  
---  
  
**End of output**  
  
# insonmnia/node/services.go  
# Package / Component    
**node**  
  
## Imports    
```go  
import (  
	"context"  
  
	"github.com/sonm-io/core/optimus"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util/rest"  
	"golang.org/x/sync/errgroup"  
	"google.golang.org/grpc"  
)  
```  
  
## External Data / Input Sources    
| Source | Description |  
|--------|-------------|  
| `remoteOptions` | Configuration struct passed to all constructor functions (`newInterceptedAPI`, `newMarketAPI`, …). It also contains fields used by `optimus.NewPredictorService`. |  
| `optimus.PredictorService` | Service that performs order prediction; created in `newServices`. |  
  
## TODOs    
No explicit TODO comments are present in this file.  
  
---  
  
# Summary of Major Code Parts    
  
### 1. `services` struct definition    
The core data holder for the node component. It aggregates all gRPC and REST services needed by the application:  
- **intercepted** – intercepting API (worker, DWH, inspect)  
- **market**, **deals**, **tasks**, **master**, **token**, **blacklist**, **profile**, **monitoring**  
- **orderPredictor** – optional predictive service  
  
### 2. `newServices` constructor    
Creates a fully‑initialized `services` instance from a given `remoteOptions`. Each field is populated by calling the corresponding *API* creation helper, and the predictor service is built via `optimus.NewPredictorService`.  
  
### 3. `RegisterGRPC` method    
Registers all services in the struct with a gRPC server:  
- Registers worker‑management, worker, DWH, inspect, market, deal‑management, task‑management, master‑management, token, blacklist, profiles, monitoring and (conditionally) order‑predictor servers.  
- Returns `nil` on success; errors are ignored for simplicity.  
  
### 4. `RegisterREST` method    
Registers the same services with a REST server:  
- Uses `server.RegisterService` for each service type, handling errors individually.  
- Includes an extra check for the optional predictor before registering it.  
  
### 5. Interceptor helpers    
Two small helper methods expose the unary and stream interceptors from the *intercepted* API to external callers.  
  
### 6. `Run` method    
Starts the order‑predictor service asynchronously using an errgroup context:  
- Launches a goroutine that calls `m.orderPredictor.Serve(ctx)`.  
- Waits for completion via `<-ctx.Done()` and returns any error from the group.  
  
---  
  
# insonmnia/node/tasks.go  
**Package / Component**    
`node`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"errors"  
	"fmt"  
	"io"  
	"strconv"  
  
	"github.com/sonm-io/core/proto"  
	"go.uber.org/zap"  
	"google.golang.org/grpc/codes"  
	"google.golang.org/grpc/metadata"  
	"google.golang.org/grpc/status"  
)  
```  
  
---  
  
### External data / input sources  
| Function | Input type | Description |  
|----------|------------|-------------|  
| `List` | `*sonm.TaskListRequest` | Request to list tasks for a deal |  
| `Start` | `*sonm.StartTaskRequest` | Request to start a new task |  
| `JoinNetwork` | `*sonm.JoinNetworkRequest` | Request to join a network on a worker |  
| `Status` | `*sonm.TaskID` | Query a single task status |  
| `Logs` | `*sonm.TaskLogsRequest`, `srv sonm.TaskManagement_LogsServer` | Stream logs from a worker |  
| `Stop` | `*sonm.TaskID` | Stop an existing task |  
| `PushTask` | `clientStream sonm.TaskManagement_PushTaskServer` | Push a new task image to a worker |  
| `PullTask` | `*sonm.PullTaskRequest`, `srv sonm.TaskManagement_PullTaskServer` | Pull a task image from a worker |  
  
---  
  
### TODOs  
- `//Deprecated, use workerAPI via interceptor` – indicates that this API is legacy and may be replaced by an interceptor‑based implementation.  
  
---  
  
## Major code parts  
  
### 1. `tasksAPI` struct    
Holds remote options (`remotes`) and a logger (`log`). It implements the `sonm.TaskManagementServer` interface.  
  
### 2. `List` method    
* Validates request, logs deal ID, obtains a worker client for the deal, fetches deal info, merges completed & running tasks into a reply map, and returns it.*  
  
### 3. `Start` method    
* Starts a task on a worker by delegating to the worker client’s `StartTask`. It simply forwards the request and returns the reply.*  
  
### 4. `JoinNetwork` method    
* Joins a network for a given task ID on a worker, building a `WorkerJoinNetworkRequest` from the incoming request.*  
  
### 5. `Status` method    
* Retrieves status of a single task by calling the worker client’s `TaskStatus`.*  
  
### 6. `Logs` streaming method    
* Streams logs from a worker to the caller: receives chunks in a loop, forwards them via the server stream until EOF.*  
  
### 7. `Stop` method    
* Stops a running task on a worker and returns an empty reply.*  
  
### 8. `PushTask` streaming method    
* Extracts metadata from the client stream (`extractStreamMeta`), obtains a worker client, starts a push stream, then loops: receives chunks from the client, sends them to the worker, tracks bytes committed, and finally forwards progress meta back to the caller.*  
  
### 9. `PullTask` streaming method    
* Similar to `PushTask`, but pulls data from a worker instead of pushing. It sets headers on the server stream and streams received chunks until EOF.*  
  
### 10. Helper: `extractStreamMeta`    
* Reads metadata keys “deal” and “size” from the incoming context, builds an outgoing context with those values, parses size into int64, and returns a `streamMeta` struct.*  
  
### 11. Constructor: `newTasksAPI`    
* Creates a new `tasksAPI` instance using provided remote options and logger.*  
  
---  
  
# insonmnia/node/tokens.go  
**Package / Component**    
`node`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"fmt"  
  
	"github.com/ethereum/go-ethereum/crypto"  
	"github.com/sonm-io/core/insonmnia/auth"  
	"github.com/sonm-io/core/proto"  
)  
```  
The file pulls in the standard `context` and `fmt` packages, plus three external libraries:  
* `github.com/ethereum/go-ethereum/crypto` – for address conversion.  
* `github.com/sonm-io/core/insonmnia/auth` – to extract wallet data from a context.  
* `github.com/sonm-io/core/proto` – contains the protocol buffer types used throughout.  
  
---  
  
### External Data / Input Sources  
| Type | Source | Notes |  
|------|--------|-------|  
| `remoteOptions` | defined elsewhere in the same package | holds remote configuration (e.g. key, eth client) |  
| `sonm.Empty` | proto package | empty request/response type |  
| `sonm.BalanceReply` | proto package | reply structure for balance queries |  
| `sonm.EthAddress` | proto package | Ethereum address wrapper |  
| `sonm.BigInt` | proto package | arbitrary‑precision integer wrapper |  
| `sonm.TokenTransferRequest` | proto package | transfer request payload |  
  
---  
  
### TODOs  
No explicit `TODO:` comments are present in the file.  
  
---  
  
## Summary of Major Code Parts  
  
#### 1. `tokenAPI` struct    
```go  
type tokenAPI struct {  
	remotes *remoteOptions  
}  
```  
* Holds a pointer to `remoteOptions`, which supplies all remote configuration needed for masterchain and sidechain interactions.  
  
#### 2. `TestTokens` method    
```go  
func (t *tokenAPI) TestTokens(ctx context.Context, _ *sonm.Empty) (*sonm.Empty, error)  
```  
* Placeholder for future token testing functionality; currently returns an error indicating it is not supported yet.  
  
#### 3. `Balance` method (deprecated wrapper)    
```go  
func (t *tokenAPI) Balance(ctx context.Context, _ *sonm.Empty) (*sonm.BalanceReply, error)  
```  
* Computes the node’s address from the public key (`crypto.PubkeyToAddress`) and forwards to `BalanceOf`.    
* Marked as deprecated; use `BalanceOf` directly.  
  
#### 4. `BalanceOf` method – core balance logic    
```go  
func (t *tokenAPI) BalanceOf(ctx context.Context, addr *sonm.EthAddress) (*sonm.BalanceReply, error)  
```  
* Calls the masterchain token contract to get live balance (`live`) and sidechain token contract for side balance (`side`).    
* Wraps both balances into a `sonm.BalanceReply` containing:  
  - Live balance (SNM)  
  - Live Ethereum balance (Eth)  
  - Side balance (SNM)  
  
#### 5. `Deposit` method    
```go  
func (t *tokenAPI) Deposit(ctx context.Context, amount *sonm.BigInt) (*sonm.Empty, error)  
```  
* Approves the masterchain token for a given allowance and then pays in via the masterchain gate.  
* Returns an empty response on success.  
  
#### 6. `Withdraw` method    
```go  
func (t *tokenAPI) Withdraw(ctx context.Context, amount *sonm.BigInt) (*sonm.Empty, error)  
```  
* Approves the sidechain token for a given allowance and then pays in via the sidechain gate.  
* Returns an empty response on success.  
  
#### 7. `MarketAllowance` method    
```go  
func (t *tokenAPI) MarketAllowance(ctx context.Context, _ *sonm.Empty) (*sonm.BigInt, error)  
```  
* Extracts a wallet address from the context using `auth.ExtractWalletFromContext`.    
* Queries the sidechain token contract for allowance toward the market address and returns it as a `sonm.BigInt`.  
  
#### 8. `Transfer` method    
```go  
func (t *tokenAPI) Transfer(ctx context.Context, request *sonm.TokenTransferRequest) (*sonm.Empty, error)  
```  
* Executes a transfer on the sidechain token contract to the recipient specified in the request.  
* Returns an empty response upon success.  
  
#### 9. `newTokenManagementAPI` constructor    
```go  
func newTokenManagementAPI(opts *remoteOptions) sonm.TokenManagementServer {  
	return &tokenAPI{remotes: opts}  
}  
```  
* Creates a new instance of `tokenAPI`, wiring it with the supplied remote options.  
* Returns an implementation of the `sonm.TokenManagementServer` interface.  
  
---  
  
All methods return either a concrete response type or an error, following Go idioms for RPC‑style APIs. The file is ready to be integrated into the larger node package and will later contribute to the overall token management service.  
  
# insonmnia/node/worker.go  
**Package & Imports**    
- **package name:** `node`    
- **Imports:**  
  ```go  
  "context"  
  "fmt"  
  "io"  
  "reflect"  
  "strings"  
  
  "github.com/ethereum/go-ethereum/crypto"  
  "github.com/sonm-io/core/insonmnia/auth"  
  "github.com/sonm-io/core/proto"  
  "github.com/sonm-io/core/util"  
  "github.com/sonm-io/core/util/xgrpc"  
  "go.uber.org/zap"  
  "golang.org/x/sync/errgroup"  
  "google.golang.org/grpc"  
  "google.golang.org/grpc/codes"  
  "google.golang.org/grpc/metadata"  
  "google.golang.org/grpc/status"  
  ```  
  
**External Data / Input Sources**    
- `remoteOptions` – passed to the constructor and used throughout for remote client creation.    
- Metadata keys:    
  - `util.WorkerAddressHeader` – contains a worker address in unary requests.    
  - `"deal"` – holds a deal ID for worker client look‑ups.    
- gRPC method names are parsed via `xgrpc.ParseMethodInfo`.  
  
**TODOs**    
| # | Description |  
|---|--------------|  
| 1 | Deduplicate logic inside `streamIntercept` (currently duplicated code paths). |  
| 2 | Add a “CloseAndRecv” handling for the third case in the streaming loop. |  
  
---  
  
## Summary of Major Code Parts  
  
### 1. `interceptedAPI` struct    
Defines a composite server that implements several gRPC interfaces (`WorkerServer`, `WorkerManagementServer`, `DWHServer`, `InspectServer`). It also holds a pointer to `remoteOptions` and a logger.  
  
### 2. Unary helpers    
  
| Function | Purpose |  
|----------|---------|  
| `getWorkerAddr` | Reads the worker address from incoming metadata or falls back to a default derived from the remote key. |  
| `getWorkerManagementClient` | Creates a `WorkerManagementClient` using the stored options and logs the connection. |  
| `getWorkerClient` | Retrieves a `WorkerClient` for a specific deal ID found in metadata. |  
  
### 3. `intercept` – Unary interceptor    
* Dispatches based on the method name extracted from `info.FullMethod`.    
* For each supported server (`sonm.Worker`, `sonm.WorkerManagement`, `sonm.DWH`, `sonm.Inspect`) it forwards metadata, obtains a client and calls the appropriate method via reflection.    
* Uses `reflect.ValueOf` to call the method dynamically and returns the first return value along with any error.  
  
### 4. Helper functions for reflective calls    
  
| Function | Role |  
|----------|------|  
| `callMethod` | Generic wrapper that invokes a named method on an interface using reflection. |  
| `callErrMethod` | Convenience wrapper expecting one return value (error). |  
| `callBinMethod` | Wrapper expecting two return values (result + error). |  
| `newMethodArgValue` | Creates a new argument value for a given method position, used when building streaming calls. |  
  
### 5. `streamIntercept` – Streaming interceptor    
* Similar dispatch logic as `intercept`, but handles both client and server streams.    
* Builds the request message via reflection (`newMethodArgValue`) and receives it with `ss.RecvMsg`.    
* Calls a binary method that returns a streaming client, then uses an `errgroup.Group` to concurrently handle sending and receiving on the stream.    
* TODO: deduplicate the two branches for client/server streams; TODO: add close/receive logic for the third case.  
  
### 6. Constructor    
  
```go  
func newInterceptedAPI(opts *remoteOptions) *interceptedAPI {  
    return &interceptedAPI{  
        remotes: opts,  
        log:     opts.log,  
    }  
}  
```  
  
Creates a fully initialized `interceptedAPI` instance ready to be used as a gRPC server.  
  
---  
  
