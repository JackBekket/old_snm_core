# `blockchain`

The **`blockchain`** package is a lightweight Go library that wraps an Ethereum‑based side‑/master‑chain node.  
It exposes a set of high‑level APIs (market, gatekeeper, blacklist, profile registry, etc.) and provides a client capable of reading receipts, sending transactions, and polling logs.  The code is organized around three layers:

1. **Configuration** – `config.go`, `options.go` and the constants in `addresses.go`/`gasLimits.go`.  
2. **Core logic** – `api.go`, `api_ext.go`, `client.go` and the helper functions in `util.go`.  
3. **Data types & helpers** – `types.go`, `topics.go`, `unit.go` and the test files.

The package is intended to be used as a library; it can also be built into a CLI via the scripts in `source/scripts/`.

---

## File structure

```
blockchain/
├─ addresses.go
├─ api.go
├─ api_ext.go
├─ api_ext_test.go
├─ client.go
├─ client_test.go
├─ config.go
├─ gasLimits.go
├─ options.go
├─ topics.go
├─ types.go
├─ types_test.go
├─ unit.go
└─ util.go
```

---

## Environment variables / configuration

| Variable | Default value (from code) | Purpose |
|----------|---------------------------|---------|
| `defaultMasterchainEndpoint` | string URL for master‑chain RPC | used by `WithMasterchainEndpoint` |
| `defaultSidechainEndpoint` | string URL for side‑chain RPC | used by `WithSidechainEndpoint` |
| `defaultContractRegistryAddr` | hex address of the contract registry | used by `WithContractRegistry` |
| `defaultMasterchainGasPrice` | *big.Int* value | gas price for master‑chain transactions |
| `defaultSidechainGasPrice` | *big.Int* value | gas price for side‑chain transactions |
| `defaultBlockConfirmations` | int64 | number of confirmations to wait before a block is considered final |
| `defaultLogParsePeriod` | time.Duration | interval between log‑parsing runs |
| `defaultMasterchainGasLimit` | uint64 | gas limit for master‑chain blocks |
| `defaultSidechainGasLimit` | uint64 | gas limit for side‑chain blocks |
| `defaultBlockBatchSize` | uint64 | number of blocks to batch when fetching data |

These values are read by the functional options in **options.go** and can be overridden via the corresponding `With…` functions.

---

## Flags / command‑line arguments

The package itself has no explicit CLI flags, but it is built and tested through the helper scripts:

* `source/scripts/dev.sh` – builds the library (via `make`) and runs tests.  
* `source/scripts/test.sh` – runs unit tests for the API.  

If you want to run a small demo program, add a `main.go` that imports `blockchain`, creates an instance with `NewAPI(...)`, and calls the exposed methods.

---

## Edge cases / launch scenarios

| Scenario | How to start |
|----------|--------------|
| **Build & test** | Run `source/scripts/dev.sh`.  This will compile all files, run tests in `api_ext_test.go` and `client_test.go`, and produce a binary in the current directory. |
| **Run a demo** | Add a `main.go` that imports `blockchain`, creates an API instance with default options (`WithDefaultOptions()`), then call e.g. `api.MarketAPI.OpenDeal(...)`.  The script `source/scripts/test.sh` can be used to run the test suite. |
| **Deploy contracts** | Use the JSON files in `source/deployed/contracts/…` as deployment artifacts; the client will read them via the registry address defined in `addresses.go`. |

---

## Relations between code entities

* **`addresses.go`** – defines constants that are used by *all* other files to look up contract addresses and keys.  
* **`config.go`** – holds a `Config` struct that is populated from YAML/JSON via `UnmarshalYAML`; the values are fed into the functional options in **options.go**.  
* **`client.go`** – implements `CustomEthereumClient`, which is used by all APIs to read receipts and send transactions.  The client is created lazily inside `chainOpts.getClient()`.  
* **`api.go`** – declares the high‑level interfaces (`ProfileRegistryAPI`, `EventsAPI`, etc.) and a concrete type `BasicAPI` that wires them together.  It also contains the registry logic (`setupContractRegistry`) that reads all contract addresses from the on‑chain hash map.  
* **`api_ext.go`** – extends the market API with helper methods such as `OpenDeal`.  It uses the other APIs (profile, blacklist) to gather data before calling the underlying market contract.  
* **`util.go`** – provides low‑level helpers (`extractAddress`, `WaitTxAndExtractLog`) that are used by the APIs for log extraction and receipt handling.  
* **`topics.go`** – defines all event topic constants; these are referenced in the API code when building filter queries.  
* **`types.go`** – contains data structures (e.g. `DealOpenedData`, `OrderPlacedData`) that are marshalled/unmarshalled by the APIs.  The helper type `Unit` from **unit.go** supplies common Ethereum denominations used for gas price calculations.

---

## Summary of logic

1. **Configuration**  
   * `config.go` reads a YAML file into a `Config`.  
   * Functional options in **options.go** allow overriding defaults (gas prices, endpoints, batch size).  

2. **Client**  
   * `CustomClient` wraps an `ethclient.Client` and exposes convenience methods (`GetLastBlock`, `GetTransactionReceipt`).  

3. **Registry & APIs**  
   * `BasicContractRegistry.setup()` reads all contract addresses from the on‑chain hash map using keys defined in **addresses.go**.  
   * `BasicAPI.NewAPI(...)` creates a full API instance, wiring together market, gatekeeper, blacklist, profile registry, etc.  

4. **Market logic**  
   * The extended market API (`niceMarketAPI`) orchestrates opening deals by fetching orders, masters, and profiles concurrently (via errgroup) before calling the underlying contract.  

5. **Utilities**  
   * `util.go` contains helpers for log extraction, receipt polling, and address extraction from logs.  

6. **Testing**  
   * The test files (`api_ext_test.go`, `client_test.go`, `types_test.go`) verify that the APIs behave correctly and that the client can parse receipts.

---

## Unclear / dead code

No obvious dead code was detected; all functions are referenced by at least one other file or a test.  If you add new options, remember to update **options.go** accordingly.

---