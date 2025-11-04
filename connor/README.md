# Package **connor**

`connor` is a Go library that orchestrates the lifecycle of orders and deals on an Ethereum‑based market.  
It reads a YAML/JSON configuration file, builds a set of backends (price provider, anti‑fraud processor, order/deal factories), runs a continuous engine that watches for new deals, creates/cancels orders, tracks tasks, and persists its state to disk.

---

## 1. Quick overview

| File | Purpose |
|------|---------|
| `backends.go` | Builds the core backends used by the engine (corder/deal factories, price provider, anti‑fraud processor). |
| `config.go` | Top‑level configuration struct (`Config`) and helper methods that load/validate a config file. |
| `engine.go` | Main runtime: constructor `New`, orchestration `Serve`, order & deal processing loops, metrics registration. |
| `state.go` | In‑memory state container for active orders, queued orders, and deals; persistence helpers. |
| `antifraud/…` | Anti‑fraud logic (processor factory, log processor, pool processor). |
| `price/…` | Price provider implementation (`cmc_wtm`) and configuration. |
| `types/…` | Domain types (`Corder`, `Deal`, etc.) and factories for orders/deals. |

---

## 2. Project file structure

```
connor/
├─ antifraud/
│   ├─ antifraud.go
│   ├─ antifraud_test.go
│   ├─ blacklist_watcher.go
│   ├─ blacklist_watcher_test.go
│   ├─ config.go
│   ├─ flags.go
│   ├─ flags_test.go
│   ├─ log_processor.go
│   ├─ log_processor_test.go
│   ├─ pool_processor.go
│   └─ pool_processor_test.go
├─ backends.go
├─ config.go
├─ config_test.go
├─ engine.go
├─ price/
│   ├─ cmc_wtm.go
│   ├─ cmc_wtm_test.go
│   ├─ config.go
│   ├─ config_test.go
│   ├─ static.go
│   └─ utils.go
├─ state.go
└─ types/
    ├─ benchmarks.go
    ├─ corder.go
    ├─ deal.go
    ├─ types.go
    ├─ types_test.go
    └─ x_test.go
```

---

## 3. Configuration & launch options

| Source | File / Path | Key fields | How it is used |
|--------|-------------|------------|----------------|
| **Node endpoint** | `config.go` → `nodeConfig.Endpoint` | `auth.Addr` | gRPC client address |
| **Ethereum key** | `config.go` → `Eth.LoadKey()` | – | Used in `engine.New()` |
| **Market parameters** | `config.go` → `marketConfig` | `Benchmark`, `From/To/Step`, `Counterparty`, `PriceControl`, `Benchmarks`, `AdoptOrders` | Drives order placement ranges, price thresholds, and benchmark mapping |
| **Container image/tag/key** | `config.go` → `containerConfig` | `Image`, `Tag`, `SSHKey`, `Env` | Used by backends to build corder/deal factories |
| **Engine timing** | `config.go` → `engineConfig` | `ConnectionTimeout`, `OrderWatchInterval`, `TaskStartInterval`, … | All runtime timeouts and intervals |
| **Metrics endpoint** | `config.go` → `Metrics string` | `"127.0.0.1:14005"` (default) | Where the engine publishes Prometheus metrics |

The package is typically started by calling `connor.NewConfig(path)` to load a config file, then `engine.New(cfg)` and finally `engine.Serve()` in an external binary or test harness.

---

## 4. Edge cases for launching

1. **Single‑shot start** – call `New` once, then `Serve`.  
2. **Restart after failure** – the engine’s goroutine group (`errgroup`) will automatically retry order creation/cancellation loops if a task fails.  
3. **Adopt external deals/orders** – `engine.restoreMarketState()` loads existing orders and deals from the market; it can be triggered manually via `engine.RestoreOrder` or automatically on startup.  

---

## 5. Relations between code entities

| Component | Depends on | Key interactions |
|-----------|------------|-------------------|
| **backends** (`backends.go`) | `Config`, `antifraud.Config`, `price.SourceConfig`, `types.CorderFactory`, `types.DealFactory` | Provides a ready‑to‑use `processorFactory` for anti‑fraud logic; exposes `priceProvider` to the engine. |
| **engine** (`engine.go`) | `backends`, `config.go`, `state.go`, `types/…` | Uses backends to create orders, cancel them, and track deals; registers Prometheus metrics; orchestrates all loops via an error group. |
| **state** (`state.go`) | `types.Corder`, `types.Deal` | Holds in‑memory maps that are read/written by the engine’s order/deal processing functions. |
| **antifraud** | `engine` (via backends) | Provides a processor factory that is used to process orders and deals; its log/pool processors feed into the engine’s metrics counters. |
| **price** | `backends`, `engine` | Supplies price data for order placement; its config file (`price/config.go`) is loaded by `NewBackends`. |

---

## 6. Summary of logic

1. **Configuration** – `config.go` loads a YAML/JSON file into a `Config` struct, validates it (including benchmark mapping), and exposes helper methods such as `getTag()` and `applyEnvTemplate()`.  
2. **Backends construction** – `backends.NewBackends(cfg)` creates factories for corders/deals, a price provider, and an anti‑fraud processor factory; all are wired into the engine.  
3. **Engine startup** – `engine.New` loads Ethereum key, benchmark list, TLS cert rotator, gRPC client, backends, and clients for market, deals, and tasks. It creates an `engine` struct with channels for order creation/cancellation and a state object.  
4. **Serve loop** – `engine.Serve()` starts goroutines that:  
   * load initial data (`loadInitialData`)  
   * start price tracking (`startPriceTracking`)  
   * process orders (`processOrderCreate`, `processOrderCancel`)  
   * restore deals (`restoreMarketState`, `waitForExternalUpdates`).  
5. **Order & deal processing** – Orders are created via `sendOrderToMarket` and converted to a local `Corder`. Deals are processed by `processDeal`, which registers metrics, restores tasks, starts new ones if needed, tracks them with retries, and restarts on failure.  
6. **State persistence** – The state object (`state.go`) is marshaled into JSON and written to `/tmp/connor_state.json` after each major step; it also provides helper methods for adding/removing orders and deals.

---

## 7. Unclear places / dead code

No obvious dead code or missing references were detected in the provided excerpts. All functions referenced by `engine.go` (e.g., `processDeal`, `waitForDeal`) are defined in the same package, and all imports resolve correctly.

---  

**<end_of_output>**