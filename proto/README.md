# Package **sonm**

## Project structure

```
proto/
├── ask_plan.go
├── ask_plan.pb.go
├── ask_plan.proto
├── ask_plan_test.go
├── benchmarks.pb.go
├── benchmarks.proto
├── bigint.go
├── bigint.pb.go
├── bigint.proto
├── bigint_test.go
├── capabilities.go
├── capabilities.pb.go
├── capabilities.proto
├── capabilities_test.go
├── container.go
├── container.pb.go
├── container.proto
├── dwh.go
├── dwh.pb.go
├── dwh.proto
├── geoip.pb.go
├── geoip.proto
├── github.com/prometheus/client_model/go/metrics.pb.go
├── github.com/prometheus/client_model/metrics.proto
├── gpu_ctl.pb.go
├── gpu_ctl.proto
├── gpu_device.go
├── gpu_device_test.go
├── init.pb.go
├── init.proto
├── insonmnia.go
├── insonmnia.pb.go
├── insonmnia.proto
├── insonmnia_test.go
├── inspect.pb.go
├── inspect.proto
├── log_reader.go
├── marketplace.go
├── marketplace.pb.go
├── marketplace.proto
├── marketplace_test.go
├── net.go
├── net.pb.go
├── net.proto
├── node.go
├── node.pb.go
├── node.proto
├── optimus.go
├── optimus.pb.go
├── optimus.proto
├── pty.pb.go
├── pty.proto
├── relay.go
├── relay.pb.go
├── relay.proto
├── rendezvous.go
├── rendezvous.pb.go
├── rendezvous.proto
├── tc.pb.go
├── tc.proto
├── timestamp.go
├── timestamp.pb.go
├── timestamp.proto
├── volume.pb.go
├── volume.proto
├── worker.go
├── worker.pb.go
├── worker.proto
└── worker_test.go
```

## Short summary of the package

`sonm` is a Go‑centric orchestration layer that models and manages distributed tasks, deals, orders, GPU devices, network specs, and related metrics.  
* **Data model** – Protobuf definitions (e.g. `ask_plan.proto`, `bigint.proto`, etc.) generate Go structs for CPU/GPU/RAM/Storage/Network resources (`AskPlanResources`), GPU device descriptors (`GPUDevice`), node & worker metadata, and a handful of RPC services (`WorkerManagement`, `Marketplace`, `Relay`, `Rendezvous`, etc.).  
* **Custom logic** – Hand‑written helpers provide YAML/JSON marshaling, validation, arithmetic on resources, conversion to Docker container configs, and price calculations.  
* **RPC plumbing** – Generated gRPC stubs expose services for workers, markets, relay clusters, and QOS (QoS) shaping; the client/server interfaces are fully wired in the `*.pb.go` files.

## Environment variables / flags / command‑line arguments

| Variable | Purpose |
|----------|---------|
| `SONM_LOG_LEVEL` | Optional log level for worker logs (used by `log_reader`). |
| `SONM_GRPC_PORT` | Port on which the gRPC server listens. |
| `SONM_DB_URL` | Connection string for the underlying database (used in `node.go`). |

*No explicit command‑line flags are defined; the package is intended to be started as a Go binary that registers all services.*

## Edge cases of launching

1. **Server mode** – Run `go run ./cmd/sonm_server` (or similar) to start the gRPC server that implements all services (`WorkerManagement`, `Marketplace`, etc.).  
2. **Client mode** – The same binary can act as a client by calling the generated RPC stubs; e.g., `client := NewWorkerManagementClient(conn)` and then `client.Status(ctx, ...)`.  
3. **CLI helper** – A small CLI wrapper (`cmd/sonm_cli.go`) could be added to expose sub‑commands for starting workers or querying deals.

## Relations between code entities

| Entity | Related types / functions |
|--------|---------------------------|
| `AskPlanCPU` | Serialized by `AskPlanCPU.MarshalYAML` / `UnmarshalYAML`; used inside `AskPlanResources`. |
| `GPUDevice` | Has a `FillHashID()` that relies on `structhash.Md5`; the hash is later read by `TypeFromVendorID`. |
| `Node` | Holds a list of `Worker`s; each worker exposes its own RPC interface. |
| `Rendezvous` | Provides network discovery; its `PublishRequest` and `ConnectRequest` are used by `RelayClusterReply`. |
| `Timestamp` | Wraps a Unix timestamp; used in many request/response structs (e.g., `GetDealInfo`). |

The protobuf files (`*.pb.go`) register all types with the runtime, while the hand‑written files provide business logic. The tests (`*_test.go`) confirm that YAML/JSON marshaling and validation work as expected.

## Summary of what the package does

`sonm` models a distributed task scheduler:  
* **Ask plans** describe CPU/GPU/RAM/storage/network requirements for a deal.  
* **GPU devices** are hashed, keyed by vendor ID, and used to build cgroup resources.  
* **Nodes & workers** expose gRPC services that allow starting/stopping tasks, querying deals, and pushing logs.  
* **Relay / rendezvous** handle network topology discovery and cluster membership.  
* **Marketplace** aggregates orders and deals; the RPC layer allows creating orders, fetching deals, and managing blacklists.  

All of this logic is tied together by protobuf‑generated types and custom helper methods that perform arithmetic, marshaling, and validation.

---