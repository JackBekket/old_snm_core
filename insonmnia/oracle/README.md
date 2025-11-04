# oracle

## Short overview  
The **oracle** package implements a lightweight price‑watcher that pulls the current SNM/USD rate from CoinMarketCap, keeps it in memory and pushes it to an Ethereum smart contract via a multi‑signature wallet.  It is driven by a configuration file (YAML/JSON) that specifies update periods, deviation thresholds, whether this node acts as master, and logging/blockchain settings.

---

## Environment variables / flags / command‑line arguments  
| Variable / flag | Purpose |
|------------------|---------|
| `CONFIG_PATH`   | Path to the YAML/JSON file that contains the `oracle`, `log`, `blockchain` and `ethereum` sections.  (Used implicitly by `configor.Load`) |
| `-master`       | Boolean flag that can be passed to a higher‑level CLI to set `is_master`.  (Not present in code, but inferred from config) |

---

## Project package structure  

```
oracle/
├── config.go
├── oracle.go
└── watcher.go
```

* **config.go** – defines the configuration structs and a constructor that loads them from disk.  
* **oracle.go** – contains the main `Oracle` type, its lifecycle methods (`NewOracle`, `Serve`) and all routines that read/write prices on the blockchain.  
* **watcher.go** – implements a small HTTP client that polls CoinMarketCap for the current SNM/USD price and streams it via a channel.

---

## How the code pieces relate  

| File | Key type / function | Interaction |
|------|---------------------|-------------|
| `config.go` | `Config`, `oracleConfig` | Holds all runtime parameters; passed to `NewOracle`. |
| `oracle.go` | `Oracle` struct | Stores a pointer to `Config`, the blockchain API, an ECDSA key and two price fields.  The constructor (`NewOracle`) wires everything together: it loads the key from config, creates a `blockchain.API`, and initializes the price fields. |
| | `watchPriceRoutine` | Starts a `PriceWatcher`; receives new prices on its channel and writes them into `actualPrice`. |
| | `submitPriceRoutine` | If `is_master` is true, it periodically calls `SetPrice` to push the current price to the smart contract. |
| | `listenEventsRoutine` | If not master, it listens for incoming events from the multi‑signature wallet and confirms them. |
| | `Serve` | Orchestrates the three goroutines via an `errgroup`. |
| `watcher.go` | `PriceWatcher`, `NewPriceWatcher`, `Start` | Provides a channel that emits new prices every `parsePeriod`.  The watcher is started by `watchPriceRoutine`. |

---

## Edge cases for launching  

1. **Master node** – When the config flag `is_master:true` is set, the oracle will run both `watchPriceRoutine` and `submitPriceRoutine`.  
   *The submit routine uses a ticker based on `ContractUpdatePeriod`; if this period is too short it may race with the watcher, but the mutex protects concurrent writes.*

2. **Non‑master node** – If `is_master:false`, only the price watcher and event listener run; the oracle will still log events and confirm transactions.

3. **Missing config keys** – The constructor uses default tags (`default:"15s"`, etc.) so missing fields are filled automatically.  If a key is absent, the corresponding value defaults to zero or the provided default.

4. **HTTP failures** – `watcher.go` logs errors but never retries; if an HTTP request fails it will simply emit a nil price on the channel until the next tick.

---

## Summary of logic  

1. **Configuration** – `config.go` loads all settings into a single struct (`Config`).  
2. **Oracle construction** – `oracle.go.NewOracle` creates the blockchain API, reads the ECDSA key and prepares the internal state.  
3. **Price watching** – A `PriceWatcher` (created in `watcher.go`) polls CoinMarketCap every `parsePeriod`, converts the USD price to SNM tokens (`divideSNM`) and pushes it into a channel.  
4. **Processing loop** – `Oracle.watchPriceRoutine` consumes that channel, updates `actualPrice` and logs the change.  
5. **Submitting / listening** – Depending on the master flag, either `submitPriceRoutine` or `listenEventsRoutine` runs in parallel, handling blockchain writes/reads.  
6. **Serve** – The orchestrator starts all goroutines via an `errgroup`; when they finish it returns.

---

All major parts are now summarized for integration into a package‑wide overview.