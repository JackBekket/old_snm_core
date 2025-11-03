# Oracle Package Summary

The `oracle` package implements an on-chain price oracle for Sonm (SNM) tokens. It fetches SNM prices from CoinMarketCap, validates them against configurable thresholds, and submits updates to a smart contract (`OracleUSD`) if running as the master node. The core logic revolves around continuous price monitoring, submission routines triggered by time intervals, and event listening for transaction confirmations. Configuration is loaded from YAML files using `configor`, allowing customization of blockchain connections, Ethereum keys, update periods, and deviation limits.

**Configuration:**

*   **Config File Path:** Specified via the `path` argument in `NewConfig`.
*   **Environment Variables:** Influence default values if not overridden in the config file (not explicitly listed but implied by `configor`).
*   **Blockchain API (`blockchain.API`)**: Configured through `Config.Blockchain`.
*   **Ethereum Private Key Path:** Defined within `Config` for signing transactions.
*   **Price Update Period:** Controlled by `oracleConfig.PriceUpdatePeriod`.
*   **Contract Update Period:** Managed via `oracleConfig.ContractUpdatePeriod`.
*   **Deviation Threshold (`Percent`)**: Configured in `oracleConfig` to validate price submissions.

**Launch Edge Cases:**

The application can be launched as either a master node (submitting prices) or an event listener (validating transactions). The `IsMaster` flag within the configuration determines this behavior. If running as the master, it requires valid Ethereum credentials and blockchain connectivity. Event listeners only need blockchain access to monitor contract events.  If no config file is provided, default values are used, potentially leading to incorrect operation if not properly configured.

**Project Package Structure:**

```
insonmnia/oracle/
├── config.go       # Configuration loading and structures
├── oracle.go       # Core Oracle logic (price monitoring, submission)
└── watcher.go      # Price fetching from CoinMarketCap API
```

**Code Relations & Potential Issues:**

The `watcher.go` component directly depends on the external CoinMarketCap API; any downtime or changes to this API will break price updates. The `oracle.go` relies heavily on blockchain interactions via the `blockchain.API`; incorrect configuration here can lead to failed transactions.  The mutex in `oracle.go` protects shared state, but improper locking could still cause race conditions under heavy load. No dead code was identified during this analysis.