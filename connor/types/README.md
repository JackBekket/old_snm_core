# Package `types`

The **connor/types** package implements a thin abstraction layer over the Sonm‑IO protobuf structures.  
It provides helpers to create and manipulate *orders* (`Corder`), *deals* (`Deal`) and to diff two sets of orders into *create / restore / cancel* actions.

---

## File structure

```
connor/types/
├── benchmarks.go
├── corder.go
├── deal.go
├── types.go
├── types_test.go
└── x_test.go
```

All files belong to the same `types` package and are compiled together.  
The test files exercise the public API; they can be run with `go test ./connor/types`.

---

## Environment variables, flags & command‑line arguments

| Variable / Flag | Purpose |
|-----------------|---------|
| `SONM_MIN_NUM_BENCHMARKS` (used in tests) | Size of the benchmark slice (`sonm.MinNumBenchmarks`). |
| `-run` (go test flag) | Run specific tests, e.g. `go test -run TestDivideOrders ./connor/types`. |

No explicit command‑line arguments are defined inside the package; it is intended to be used as a library.

---

## Summary of code logic

### 1. `benchmarks.go`

* **Type alias** – `type Benchmarks sonm.Benchmarks` gives local access to the protobuf struct.
* **Constructor** – `newZeroBenchmarks()` creates an empty slice with length `sonm.MinNumBenchmarks`.
* **GPU setters** – `setGPUEth`, `setGPUZcash`, `setGPURedshift` write values into indices 9‑11 of the underlying slice.
* **Unwrap helper** – converts the alias back to a pointer to the original struct (`*sonm.Benchmarks`) so that other methods can call its getters.
* **Map conversion** – `toMap()` builds a map from string keys (e.g. `"gpu-eth-hashrate"`) to benchmark values; it is used by the wrapper function `benchmarksToMap`.

### 2. `corder.go`

* **Factory interface** – `CorderFactory` defines three ways to create a `Corder`: from an existing order, from raw parameters, or from a slice of orders.
* **Concrete implementation** – `anyCorderFactory` stores the benchmark index, tag string and counterparty address; it implements all three methods.
* **Core type** – `Corder` embeds a pointer to `sonm.Order` and remembers which benchmark index it uses.  
  Methods:
  * `GetHashrate()` – reads the hashrate from the stored benchmarks.
  * `AsBID()` – expands the order into a full `sonm.BidOrder`, filling all nested fields (price, blacklist, resources, etc.).
  * `RestorePrice()` – recomputes the original price by dividing the current price by the benchmark value.
  * `IsReplaceable(actualPrice *big.Int, delta float64)` – decides whether an incoming order should replace the existing one.
  * `Hash()` – creates a SHA‑1 hash string from selected fields (benchmarks, counterparty, netflags).

* **Cancel tuple** – `CorderCancelTuple` holds a reference to a `Corder` and a delay duration; it is used by higher‑level logic that schedules order cancellations.

### 3. `deal.go`

* **Factory interface & implementation** – analogous to the order factory but for deals.
* **Core type** – `Deal` embeds a pointer to `sonm.Deal` and stores a benchmark index.
* **Accessor helpers** – `BenchmarkValue()`, `Unwrap()`, `RestorePrice()` and `IsReplaceable()` provide convenient arithmetic on deal data.

### 4. `types.go`

* **Task status wrapper** – `TaskStatus` embeds the protobuf reply struct and adds an `ID` field.
* **Order set abstraction** – `OrdersSets` holds three slices (`Create`, `Restore`, `Cancel`) of pointers to `Corder`.
* **Diff logic** – `DivideOrdersSets(existing, target []*Corder) *OrdersSets` builds lookup maps keyed by hashrate, then populates the three slices:
  * Orders that already exist and match a target order are added to `Restore`.
  * New orders (no existing match) go into `Create`.
  * Orders present in the existing set but missing from the target are queued for cancellation.

---

## Relations between code entities

| Entity | Depends on | Notes |
|--------|------------|-------|
| `Benchmarks` | `sonm.Benchmarks` | Provides slice access; used by `CorderFactory`. |
| `CorderFactory` | `Benchmarks`, `DealFactory` | Creates orders from raw data or existing structs. |
| `Corder.AsBID()` | `Benchmarks`, `DealFactory` | Expands an order into a full bid‑order; this is the main entry point for higher‑level logic. |
| `DivideOrdersSets` | `Corder` | Operates on slices of orders to produce diff sets. |
| Tests (`types_test.go`, `x_test.go`) | All above | Verify that the factory, map conversion and diff logic work as expected. |

The code is largely self‑contained; there are no external dead branches or missing references.

---

## Edge cases for launching

* **Library usage** – Import `connor/types` in another package and call `NewCorderFactory(...)`, then use `DivideOrdersSets()` to compute changes before persisting them.
* **CLI entry point** – If a command‑line tool is added, it could accept flags such as:
  * `-tag string` – counterparty tag for orders,
  * `-bench int` – benchmark index,
  * `-addr string` – Ethereum address of the counterparty.
  The tool would then call `NewCorderFactory(tag, bench, common.HexToAddress(addr))`, build a set of target orders and invoke `DivideOrdersSets()`.

---

**<end_of_output>**