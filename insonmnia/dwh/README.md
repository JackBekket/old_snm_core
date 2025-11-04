# Package `dwh`

The **`dwh`** package implements a data‑warehouse service that reads blockchain events, stores them in PostgreSQL and exposes a gRPC/HTTP API.  
It is split into three logical layers:

| Layer | Files | Key responsibilities |
|-------|-------|------------------------|
| **Configuration** | `config.go` | Load YAML config, expose `DWHConfig`, helper structs for storage & processor configuration. |
| **Processor** | `processor.go` | L1 event watcher that pulls market events from the blockchain API and writes them to the DB. |
| **Server** | `server.go` | gRPC/HTTP server that serves RPC calls defined in `core/proto`. |

The test harness (`*_test.go`) exercises all public methods, while `setup_test.go` seeds a PostgreSQL instance inside Docker for integration tests.

---

## Short summary of the package

* **Configuration** – `NewDWHConfig(path string)` loads a YAML file into a `DWHConfig` struct that contains logging, gRPC/HTTP addresses, storage endpoint and other runtime options.  
* **Processor** – `L1Processor` watches for new market events, processes them in batches, updates deals, orders, profiles, validators, certificates, etc., and keeps track of the last processed event.  
* **Server** – `DWH` starts a gRPC server on `GRPCListenAddr`, an HTTP REST server on `HTTPListenAddr`, monitors statistics and sync status, and exposes RPC methods that delegate to a `sqlStorage` helper.  

The package is fully testable: the tests create mock blockchain/market APIs, seed data, run all handlers and verify results.

---

## Environment variables / configuration flags

| Variable | Source file | Description |
|----------|-------------|-------------|
| `GRPCListenAddr` | `config.go` | TCP address for gRPC server. |
| `HTTPListenAddr` | `config.go` | TCP address for HTTP server. |
| `Storage.Endpoint` | `config.go` | PostgreSQL connection string (e.g., `"postgresql://localhost:15432/template1?user=postgres&password=dwh_tester&sslmode=disable"`). |
| `NumWorkers` | `processor.go` | Number of workers for the L1 processor. |
| `ColdStart.UpToBlock` | `processor.go` | Target block number for cold‑start processing. |

Command‑line arguments are not explicitly defined in this package; the main entry point would be a separate binary that calls `NewDWH(...).Serve()` or `NewL1Processor(...).Start()`.

---

## Edge cases / launch scenarios

| Scenario | How to start |
|----------|--------------|
| **Standalone server** | Build a binary that imports `dwh` and executes: <br>`cfg, _ := dwh.NewDWHConfig("config.yaml")`<br>`srv, _ := dwh.NewDWH(context.Background(), cfg, key)`<br>`srv.Serve()` |
| **Processor only** | In tests or a separate binary: <br>`p, _ := dwh.NewL1Processor(context.Background(), cfg, key)`<br>`p.Start()` |

Both `Serve` and `Start` open the DB, create the blockchain API, and launch goroutines that keep the state in sync with the chain.

---

## Project package structure

```
insonmnia/dwh/
├─ config.go
├─ processor.go
├─ processor_test.go
├─ server.go
├─ server_test.go
├─ setup_test.go
├─ sql.go
├─ storage.go
└─ util.go
```

---

## Relations between code entities

* `DWHConfig` (config.go) feeds into both `NewDWH` and `NewL1Processor`.  
* `sqlStorage` (storage.go) is created in `setupDB` inside `processor.go` and used by all RPC methods in `server.go`.  
* The processor’s event dispatcher (`eventsDispatcher`) collects events from the blockchain API and hands them to handler functions that call into `sqlStorage`.  
* Test harnesses create mock APIs, seed data via `storage.builder()` calls, and verify that each handler updates the correct tables.  

No dead code was detected; all public methods are exercised by the tests.

---