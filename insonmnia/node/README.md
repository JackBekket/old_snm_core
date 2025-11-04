# node

## Project package structure
```
node/
├── blacklist.go
├── config.go
├── deals.go
├── market.go
├── master.go
├── mod.go
├── monitoring.go
├── options.go
├── profiles.go
├── remote.go
├── server.go
├── server_test.go
├── services.go
├── tasks.go
└── tokens.go
└── worker.go
```

---

## What the package does

`node` is a self‑contained gRPC/REST service that exposes all core Sonm‑NIA functionality (blacklist, deals, market, master, monitoring, etc.) over a single node instance.  
The code is split into logical sub‑APIs:

| File | Responsibility |
|------|----------------|
| **blacklist.go** – CRUD for the node’s blacklist. |
| **config.go** – YAML configuration loader and struct definition. |
| **deals.go** – Deal management (listing, finishing, opening, quick‑buy). |
| **market.go** – Order placement, cancellation, purging, etc. |
| **master.go** – Master‑management (workers list/confirm/remove). |
| **mod.go** – Node construction and runtime (`Node`, `Serve`). |
| **monitoring.go** – NPP metrics collection. |
| **options.go** – Functional options for the node (gRPC, REST, QUIC, SSH, etc.). |
| **profiles.go** – Profile CRUD. |
| **remote.go** – Remote client factory and helper functions. |
| **server.go** – Network listener creation, gRPC/REST registration, server lifecycle. |
| **services.go** – Aggregates all sub‑APIs into a single `Services` implementation. |
| **tasks.go** – Task CRUD and streaming logs. |
| **tokens.go** – Token balance/transfer logic. |
| **worker.go** – Interceptor that forwards worker‑level RPCs to the appropriate client. |

---

## Environment variables, flags & command‑line arguments

* **YAML config file** – `node/config.go` expects a path passed to `NewConfig(path string)`.  
  * The YAML keys are: `http_bind_port`, `bind_port`, `allow_insecure_connection`, `node`, `npp`, `log`, `blockchain`, `eth`, `dwh`, `metrics_listen_addr`, `benchmarks`, `matcher`, `predictor`, `debug`, `ssh`.  
* **Flags** – The node can be started with the following functional options (see `options.go`):  
  * `WithGRPCServer()` – enable gRPC.  
  * `WithRESTServer()` – enable REST.  
  * `WithQUIC(cfg *tls.Config)` – enable QUIC‑gRPC.  
  * `WithSSH(...)` – add an SSH proxy server.  
  * `WithLog(log *zap.Logger)` – set a custom logger.  
* **Command‑line arguments** – None are defined in this package; the node is usually started from a higher‑level main program that calls `mod.New()` and then `node.Serve(ctx)`.  

---

## How the application can be launched

1. **Configuration** – Load a YAML file into a `Config` instance:  
   ```go
   cfg, err := node.NewConfig("config.yaml")
   ```
2. **Node construction** – Build the node with optional functional options:  
   ```go
   n, err := mod.New(cfg,
       node.WithGRPCServer(),
       node.WithRESTServer(),
       node.WithQUIC(&tls.Config{...}),
       node.WithSSH(...),
   )
   ```
3. **Serve** – Run the node in a context (e.g., `context.Background()`):  
   ```go
   if err := n.Serve(context.Background()); err != nil {
       log.Fatal(err)
   }
   ```

The server creates loopback listeners for gRPC, QUIC‑gRPC and REST, registers all services via the `Services` implementation, then starts listening on each endpoint.  The node can also be launched from a test harness (see `server_test.go`) that spins up a mock server and verifies RPC connectivity.

---

## Summary of major code parts

### `blacklist.go`
* Implements `newBlacklistAPI`, `List`, `Remove` and `Purge`.  
* Uses the remote data‑warehouse (`dwh.GetBlacklist`) and Ethereum market (`eth.Blacklist().Remove`).  
* `Purge` pulls the current node address, lists all blacklist entries, then removes each entry concurrently with `xconcurrency.Run`.

### `config.go`
* Defines a `Config` struct that holds HTTP/GRPC ports, NPP config, logging, blockchain, Ethereum, DWH, metrics, benchmarks, matcher, predictor, debug and SSH settings.  
* `NewConfig(path string)` loads the YAML file into this struct.

### `deals.go`
* Provides CRUD for deals: `List`, `Status`, `Finish`, `Open`, `QuickBuy`, etc.  
* Relies on remote options (`dwh.GetDeals`, `eth.Market()`) and uses concurrency helpers to finish multiple deals in parallel.

### `market.go`
* Implements the market API: `GetOrders`, `CreateOrder`, `CancelOrder`, `Purge`/`PurgeVerbose`.  
* Uses a benchmark list from config, Ethereum crypto utilities, and a data‑warehouse client.

### `master.go`
* Provides master‑management RPCs (`WorkersList`, `WorkerConfirm`, `WorkerRemove`).  
* Wraps calls to the DWH client and Ethereum market service.

### `mod.go`
* Builds the `Node` struct that holds configuration and a gRPC/REST server.  
* `New()` creates the node, loads keys, builds remote options, assembles services, starts the server, and returns the ready instance.

### `monitoring.go`
* Exposes an NPP metrics RPC (`MetricsNPP`).  
* Uses an `npp.Dialer` from remote options to fetch raw metrics and transforms them into a protobuf reply.

### `options.go`
* Provides functional option constructors for gRPC, REST, QUIC, SSH, logging, etc.  
* The `ServerOption` type is used by `mod.New()` to configure the node.

### `profiles.go`
* CRUD for profiles: `List`, `Status`, `RemoveAttribute`.  
* Delegates to the data‑warehouse client (`dwh.GetProfiles`, `GetProfileInfo`).

### `remote.go`
* Holds a `remoteOptions` struct that bundles all remote clients (DWH, Ethereum market, NPP dialer, matcher, etc.).  
* Provides helper functions such as `getWorkerClientForDeal`, `newRemoteOptions`.

### `server.go`
* Creates loopback listeners for gRPC, QUIC‑gRPC and REST.  
* Registers services via the `Services` interface, starts listening on each endpoint, and exposes a `Serve(ctx)` method that runs all listeners concurrently.

### `services.go`
* Aggregates all sub‑APIs into a single implementation of `Services`.  
* Methods `RegisterGRPC`, `RegisterREST`, `Interceptor()`, `StreamInterceptor()` and `Run` wire the individual APIs to a gRPC/REST server.

### `tasks.go`
* Implements task CRUD (`List`, `Start`, `JoinNetwork`, `Status`, `Logs`, `Stop`, `PushTask`, `PullTask`).  
* Uses worker clients created by `remoteOptions`.

### `tokens.go`
* Provides token balance and transfer logic: `BalanceOf`, `Deposit`, `Withdraw`, `MarketAllowance`, `Transfer`.  
* Calls the masterchain and sidechain contracts via Ethereum market.

### `worker.go`
* Implements an interceptor that forwards worker‑level RPCs to the appropriate client.  
* Uses reflection to dispatch based on method name, handles unary and streaming calls, and provides helper functions for building request arguments.

---

## Edge cases

* **Empty blacklist** – `Purge` will still iterate over zero entries without error.  
* **No deals found** – `List` returns an empty reply; callers must handle the nil case.  
* **Concurrent finish** – `FinishDeals` uses a fixed concurrency level (`purgeConcurrency`) that can be tuned via a flag or config variable.  

---

**<end_of_output>**