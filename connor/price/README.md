# Price Package Summary

This package provides price fetching mechanisms from external sources like CoinMarketCap (CMC) and WhatToMine (WTM). It supports static prices as well for testing or fallback scenarios. The core logic revolves around retrieving token prices, calculating per-hash profitability based on network parameters (difficulty/nethash), and providing a consistent interface (`Provider`) for accessing these values.

**Configuration:**

*   **Environment Variables:** None explicitly used in the provided code snippets.
*   **Flags/Cmdline Arguments:** Not applicable as this appears to be a library package, not an executable.
*   **Files & Paths (Configuration):** YAML configuration files are expected for providers like CoinMarketCap and Static. The `config` package handles parsing these files.  The structure is defined in the `SourceConfig` struct with fields such as `type`, `url`, `what_to_mine_id`, and `update_interval`.
*   **Edge Cases (Launch):** Not applicable, this is a library.

**Package Structure:**

```
connor/price/
├── cmc_wtm.go       # CoinMarketCap price provider implementation
├── cmc_wtm_test.go  # Unit tests for cmc_wtm.go
├── config.go        # Configuration loading and validation logic
├── config_test.go   # Tests for configuration parsing
├── static.go        # Static price provider implementation
└── utils.go         # Utility functions (HTTP requests, data fetching)
```

**Relationships & Unclear Areas:**

The `config` package is central to initializing providers based on YAML configurations. The `cmc_wtm` and `static` packages implement the `Provider` interface defined in `config`.  The `utils` package provides reusable HTTP request logic used by both price fetchers. There are no apparent dead code sections, but TODOs suggest potential refactoring (context-aware HTTP client) and future expansion (Node-based providers). The reliance on external APIs (CoinMarketCap, WhatToMine) introduces dependencies that could affect reliability if those services become unavailable or change their API structure.