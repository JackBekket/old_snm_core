## Package: `types`

This package defines core data structures and logic for managing orders and deals within the Sonm ecosystem. It wraps protobuf definitions from `github.com/sonm-io/core/proto` with custom types like `Benchmarks`, `Corder`, and `Deal`, providing methods for manipulation, comparison, and replacement.

**Configuration:**

*   **Benchmark Indices:** The package relies on hardcoded benchmark indices (e.g., 9 for Eth, 10 for Zcash) for accessing specific benchmark values.
*   **Price Delta:** The `isDealReplaceable` function uses a `delta` value (float64) to determine if a deal should be replaced based on price differences.
*   **Cancellation Delay:** The `orderCancelMaxDelay` constant controls the maximum delay for order cancellations.

**Edge Cases:**

*   The `toMap()` function in `benchmarks.go` is marked as a "shitty crutch" and should be refactored.
*   The `DivideOrdersSets` function in `types.go` and its tests in `x_test.go` handle order replacement based on hashrate, counterparty ID, and network flags. Changes in these parameters trigger full order replacements.

**Project Structure:**

```
connor/types/
├── benchmarks.go
├── corder.go
├── deal.go
├── types.go
├── types_test.go
└── x_test.go
```

**Relationships:**

*   `Benchmarks` wraps `sonm.Benchmarks` and provides methods for setting and retrieving GPU benchmark values.
*   `Corder` wraps `sonm.Order` and manages order-specific logic like replaceability and hashing.
*   `Deal` wraps `sonm.Deal` and provides methods for retrieving benchmark values and determining deal replaceability.
*   `DivideOrdersSets` compares sets of `Corder` instances to identify orders for creation, restoration, and cancellation.