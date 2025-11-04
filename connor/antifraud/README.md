# Package **antifraud**

## Project file structure
```
connor/antifraud/
├─ antifraud.go
├─ antifraud_test.go
├─ blacklist_watcher.go
├─ blacklist_watcher_test.go
├─ config.go
├─ flags.go
├─ flags_test.go
├─ log_processor.go
├─ log_processor_test.go
├─ pool_processor.go
├─ pool_processor_test.go
└─ processor.go
```

## Environment variables, flags and command‑line arguments

| Source | Description |
|--------|-------------|
| `AllChecks` (int flag) | When set to `1`, the main loop will perform all checks in a single tick. |
| `SkipBlacklisting` (int flag) | Bitmask used by `flags.SkipBlacklist()` to decide whether blacklisting should be skipped. |
| `QualityCheckInterval` (`time.Duration`) | Interval for quality checks in `antifraud.go`. |
| `BlacklistCheckInterval` (`time.Duration`) | Interval for blacklist checks in `antifraud.go`. |
| `LogProcessorConfig.Format`, `PoolProcessorConfig.Format` | Processor format strings used by the factory. |

No explicit command‑line arguments are defined; configuration is supplied via a `Config{}` struct (see below).

## Summary of package logic

### 1. Configuration (`config.go`)
*Defines nested structs*  
- `ProcessorConfig`: common fields for log and pool processors.  
- `LogProcessorConfig` / `PoolProcessorConfig`: inline extensions with processor‑specific fields.  
- `Config`: top‑level struct that aggregates quality metrics, intervals, nested configs, and a whitelist of Ethereum addresses.

### 2. Processor abstraction (`processor.go`)
*Defines the public interface*  
```go
type Processor interface {
    Run(ctx context.Context) error
    TaskID() string
    TaskQuality() (bool, float64)
}
```
A factory (`ProcessorFactory`) creates log and pool processors based on a format string.  
`NewProcessorFactory` builds two builder functions that close over the config’s format values; `WithLogger` / `WithClientConn` supply optional logger and GRPC client.

### 3. Log processor (`log_processor.go`)
*Creates a worker that fetches logs from a node, persists them to disk and tracks hashrate.*  
- Constructor `newLogProcessor` wires the logger, config, deal, task ID and a gRPC client.  
- `Run` starts a ticker loop: every 5 s it updates an EWMA of hashrate; every second it ticks that value.  
- `fetchLogs` sends a `TaskLogsRequest`, pipes the response into a local file (`maybeOpenHistoryFile`) and parses each line with `logParser`.  
- `TaskQuality()` returns whether the warm‑up period is finished and the current hashrate relative to the benchmark.

### 4. Pool processor (`pool_processor.go`)
*Polls a mining pool (either dwarf or uleypool) and updates an EWMA of hashrate.*  
- Two constructors, `newDwarfPoolProcessor` and `newUleyPoolProcessor`, create a `commonPoolProcessor`.  
- The run loop is similar to the log processor: it ticks the EWMA every second, then triggers a fetch via either `dwarfPoolUpdateFunc` or `uleypoolUpdateFunc`.  
- Queue logic (`updateHashRateQueue`, `nonZeroHashrate`) keeps a capped deque of recent samples.

### 5. Main antifraud component (`antifraud.go`)
*Orchestrates the whole system.*  
- Holds a map of deals to their log and pool processors, a blacklist watcher map, factories, config, node connection, client and logger.  
- `Run` starts two tickers: one for quality checks (`checkDeals`) and another for blacklist checks (`checkBlacklist`).  
- `TrackTask` registers a new deal’s task by creating both processors via the factory and running them concurrently with an errgroup.  
- `DealOpened` adds a deal to the internal map and creates a watcher if needed.  
- `FinishDeal` decides which blacklist type to use, then calls `finishDealWithRetry`.  
- Helper methods (`lifeTime`, `whoToBlacklist`, `isAddressWhitelisted`) provide metrics and whitelist checks.

### 6. Blacklist watcher (`blacklist_watcher.go`)
*Keeps an address in a temporary blacklist while the deal is active.*  
- Holds logger, watched address, client, next period, un‑blacklist time and last success timestamp.  
- `Failure()` and `Success()` adjust the period and timestamps; `TryUnblacklist` removes the address from the blacklist via gRPC.

## Relations between code entities

| Entity | Depends on | Notes |
|--------|------------|-------|
| `ProcessorFactory` | `Config` (format strings) | Creates log/pool processors. |
| `logProcessor` | `commonPoolProcessor`, `blacklistWatcher` | Uses the factory to create a pool processor for each deal. |
| `commonPoolProcessor` | `logProcessor` | Provides shared logic for both dwarf and uleypool workers. |
| `antifraud` | `ProcessorFactory`, `Config`, `grpc.ClientConn` | Top‑level orchestrator that stores deals, watchers and runs checks. |
| `blacklistWatcher` | `antifraud` | Each deal has one watcher; the main loop calls its `Failure()`/`Success()`. |

The factory’s builder functions close over the format strings from `Config`, so changing a processor type (e.g., `"dwarf"` vs `"uleypool"`) automatically selects the correct constructor.

## Edge cases for launching

* **Initial startup** – `NewAntiFraud(cfg, cc)` must be called with a fully populated `Config{}` and an active gRPC connection.  
  * The first call to `TrackTask` will create both processors; they run concurrently until the context is cancelled.  
* **Quality check interval** – If `QualityCheckInterval` is too short, `checkDeals` may be called before all processors have finished their warm‑up period; the code guards against this by checking `isAddressWhitelisted`.  
* **Blacklist handling** – The watcher’s `Failure()` doubles the next period until it reaches `maxStep`; if a deal finishes early, `finishDealWithRetry` will keep retrying every 10 s.  

All tests in the package (`*_test.go`) exercise the core logic: whitelist lookup, flag parsing, log line parsing, queue handling and watcher state transitions.

---

**<end_of_output>**