# `<end_of_output>`  

## Package Overview  
**Name:** `gateway` – a lightweight IPVS‑based router that manages virtual services, backends and port allocation on the local machine.  

The package is split into platform‑specific files (`*_linux.go`, `*_nonlinux.go`) so it can be built on Linux or any other OS.  The core logic lives in `gateway_linux.go` (Linux) and `router_nonlinux.go` (other OS).  All public types are exported from the same package, making the code easy to consume.

---

## File structure  

```
insonmnia/gateway/
├─ gateway.go
├─ gateway_linux.go
├─ gateway_nonlinux.go
├─ metrics.go
├─ pool.go
├─ pool_test.go
├─ router.go
├─ router_linux.go
├─ router_nonlinux.go
```

---

## Key types & relationships  

| Type | Purpose | Notes |
|------|---------|-------|
| `ServiceOptions` (gateway.go) | Holds human‑readable config for a virtual service and internal numeric fields (`host net.IP`, `protocol uint16`). | Used by `NewServiceOptions`. |
| `RealOptions` (gateway.go) | Config for a real backend of a virtual service. | Used by `NewRealOptions`. |
| `Metrics` (metrics.go) | Aggregated counters for a service. | Simple add‑method aggregates another instance. |
| `PortPool` (pool.go) | Thread‑safe queue of free ports; maps identifiers to allocated port numbers. | Provides `Assign`, `Retain`. |
| `Gateway` (gateway_linux.go) | Core IPVS controller – holds services/backends map, a mutex and an IPVS client (`gnl2go.IpvsClient`). | Methods: `NewGateway`, `CreateService`, `CreateBackend`, `RemoveService`, `RemoveBackend`, `GetMetrics`, `Close`. |
| `Router` interface (router.go) | Abstract router that can register/deregister services, fetch metrics and close. | Implemented by `directRouter` in `router_nonlinux.go` and by `ipvsRouter` in `gateway_linux.go`. |
| `directRouter` (router_nonlinux.go) | Minimal stub implementation of `Router`. | Only `Register` is implemented; others are placeholders. |
| `ipvsRouter` (gateway_linux.go) | Full Linux‑specific router that uses the IPVS client and a port pool. | Holds maps for metrics, services, and a mutex. |
| `directVirtualService` (router_nonlinux.go) | Concrete implementation of `VirtualService`. | Provides `AddReal`, `RemoveReal`. |
| `ipvsVirtualService` (gateway_linux.go) | Linux‑specific virtual service that talks to the IPVS client. | Holds options and a gateway reference; implements `ID()`, `AddReal()`, `RemoveReal()`. |

The flow is:  
1. **Router** registers a new virtual service → creates a `ServiceOptions` via `NewServiceOptions`.  
2. The router assigns a port from the pool, builds an IPVS service with `gateway.CreateService`, and stores a `ipvsVirtualService` in its `services` map.  
3. A virtual service can add a backend (`AddReal`) which creates a `RealOptions`, calls `gateway.CreateBackend`, and returns a `Route`.  

---

## Environment variables, flags & command‑line arguments  

| Variable / Flag | Description |
|------------------|-------------|
| `DefaultProtocol` (const in gateway.go) | Default protocol string (“tcp”). |
| `DefaultSchedulingMethod` (const in gateway.go) | Default scheduling method (“wrr”). |
| `PlatformSupportIPVS` (const in gateway_nonlinux.go) | Boolean flag that can be toggled to enable IPVS support on non‑Linux platforms. |

The package does not expose a command‑line binary itself, but the public constructors (`NewGateway`, `newIPVSRouter`) accept a `context.Context`.  Typical usage would look like:

```go
ctx := context.Background()
gate, _ := gateway.NewGateway(ctx)
pool := pool.NewPortPool(10, 5)          // base port 10, size 5
router := gateway.newIPVSRouter(ctx, gate, pool)

svc, err := router.Register("web", "tcp")
```

---

## Edge cases & launch scenarios  

| Scenario | What to watch for |
|----------|--------------------|
| **Linux build** – `gateway_linux.go` and `router_linux.go` are compiled.  The IPVS client (`gnl2go`) must be available; otherwise the router will fail at runtime. |
| **Non‑Linux build** – `gateway_nonlinux.go` and `router_nonlinux.go` provide a minimal stub; currently only registration works, so on non‑Linux you’ll need to implement the missing methods before full functionality is achieved. |
| **Port exhaustion** – `Assign()` returns an error if the pool queue is empty; tests in `pool_test.go` cover this case.  The router should handle it gracefully by propagating the error up the call chain. |
| **Duplicate service ID** – `Register()` checks for existing IDs and will return an error if a duplicate is attempted.  The code logs the creation of the service via Uber‑Zap logger. |

---

## Summary of major logic  

1. **Service configuration** (`gateway.go`)  
   * `NewServiceOptions` resolves host IP, normalises protocol string, sets default scheduling method and returns a fully populated `ServiceOptions`.  
   * `NewRealOptions` does the same for backends.

2. **IPVS client handling** (`gateway_linux.go`)  
   * `NewGateway` creates an `ipvsClient`, calls `Init()`/`Flush()`, and returns a ready‑to‑use gateway.  
   * `CreateService` / `CreateBackend` wrap the IPVS client calls with logging and map updates.

3. **Port allocation** (`pool.go`)  
   * Randomised port queue ensures even distribution of ports across services.  
   * `Assign()`/`Retain()` provide thread‑safe allocation/release.

4. **Router orchestration** (`gateway_linux.go`)  
   * `newIPVSRouter` builds an `ipvsRouter`, wiring the gateway and pool together.  
   * `Register()` assigns a port, creates service options, calls the gateway to add the service, stores the virtual service in its map, and logs the operation.  
   * `GetMetrics()` aggregates per‑service metrics into a single `Metrics` instance.

5. **Testing** (`pool_test.go`)  
   * Validates that ports are correctly assigned, retained, and re‑assigned; covers edge cases such as double assignment or retaining an empty pool.

---

## Final remarks  

The package is ready for Linux builds; the non‑Linux stub will need to be expanded once a platform‑specific IPVS client becomes available.  The code is intentionally modular: each file focuses on one aspect (gateway, router, metrics, pool) and can be extended independently.  All public functions are protected by mutexes (`mu sync.Mutex`) for thread safety.

---