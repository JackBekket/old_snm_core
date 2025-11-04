## Package: price

**Summary:**

The `price` package provides a flexible framework for fetching cryptocurrency prices from external sources, primarily CoinMarketCap and WhatToMine. It supports static price configurations and dynamic fetching with retries. The package uses factories to create price providers from configuration, allowing for easy extension with new sources. Configuration is loaded via YAML, and the package includes validation to ensure data integrity.

**Configuration:**

*   **Environment Variables:** None explicitly defined in the code.
*   **Flags/Cmdline Arguments:** None.
*   **Files/Paths:**
    *   `config.yaml`: Expected to contain provider configurations (type, URL, price, etc.).
    *   `cmc_wtm.go`: Contains the CoinMarketCap price provider logic.
    *   `static.go`: Contains the static price provider logic.
    *   `config.go`: Defines the factory pattern for creating price providers.
    *   `utils.go`: Contains utility functions for fetching data from external APIs (CoinMarketCap, WhatToMine).
*   **Configuration Parameters:**
    *   `CoinMarketCapConfig`: Requires `WhatToMineID` and `URL`.
    *   `StaticProviderConfig`: Requires `Price`.

**Edge Cases (Launch):**

*   The package is designed to be integrated into a larger application. It does not have a standalone command-line interface.
*   Invalid YAML configurations (e.g., missing required fields, incorrect data types) will result in errors during initialization.
*   API failures (CoinMarketCap, WhatToMine) will trigger retries, but persistent failures will lead to errors.

**Package Structure:**

```
connor/price/
├── cmc_wtm.go
├── cmc_wtm_test.go
├── config.go
├── config_test.go
├── static.go
└── utils.go
```

**Relations Between Entities:**

*   `config.go` defines the `Factory` interface and concrete implementations for creating price providers.
*   `cmc_wtm.go` implements a dynamic price provider that fetches data from CoinMarketCap.
*   `static.go` implements a static price provider that reads the price from a configuration file.
*   `utils.go` provides utility functions for fetching data from external APIs (CoinMarketCap, WhatToMine).
*   `config_test.go` and `cmc_wtm_test.go` contain unit tests for validating the configuration and price provider logic.

**Unclear Places/Dead Code:**

*   The `Margin` parameter in `cmc_wtm.go` and `static.go` is not used within the provider implementations, suggesting it might be intended for a higher-level calculation or adjustment.
*   The `TODO: ctxhttp.Get()` comment in `utils.go` indicates a potential improvement to use a context-aware HTTP client for better control over requests.