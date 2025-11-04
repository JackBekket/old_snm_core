```markdown
# connor Package Summary

**Package Name:** `connor` (based on directory structure)

This package implements the core logic for a SONM-based marketplace engine, handling order creation, cancellation, deal processing, and anti-fraud measures. It interacts with external services via gRPC clients (market, deals, tasks) and uses Prometheus metrics to track active deals and orders. Configuration is loaded from files using `configor`.

**Environment Variables/Flags:**

*   Configuration file path can be specified through environment variables or command-line flags.
*   Ethereum key paths are configurable via configuration files.
*   TLS settings (certificates, keys) for gRPC clients are defined in the configuration.
*   Benchmark configurations and market parameters are loaded from external sources.

**File Structure:**

```
connor/
├── antifraud/          # Anti-fraud processing logic
│   ├── antifraud.go
│   ├── blacklist_watcher.go
│   └── ...
├── price/              # Price calculation and retrieval mechanisms
│   ├── cmc_wtm.go
│   └── ...
├── config.go           # Configuration loading and validation
├── engine.go           # Core marketplace engine logic
├── state.go            # In-memory order and deal state management
└── types/              # Custom data types (Corder, Deal)
```

**Key Components:**

*   `Config`: Main configuration struct aggregating settings for market behavior, container execution, logging, anti-fraud measures, and price sources.
*   `Engine`: Orchestrates the main loop: loading initial data, starting background goroutines for price tracking/processing, restoring market state from gRPC clients.
*   `State`: Manages active orders, queued orders, and deals in memory with mutex locking for concurrency safety. Provides persistence via JSON dumping to `/tmp/connor_state.json`.

**Edge Cases:**

*   The engine relies on external gRPC services (market, deal management, task management). Failures or unavailability of these services can lead to errors.
*   Configuration validation is critical; invalid settings may cause runtime panics or unexpected behavior.
*   Anti-fraud checks are essential for preventing malicious activity. Improper configuration could bypass security measures.

**Unclear Areas/Potential Dead Code:** None immediately apparent from the provided summaries, but thorough code review would be necessary to confirm. The reliance on external gRPC clients and complex configurations introduces potential failure points that require careful monitoring.
```