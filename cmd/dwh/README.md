# Package: dwh

This package implements a Data Warehouse (DWH) service with an L1 event processor, configured via a file and command-line arguments. It handles Ethereum key loading, concurrent execution of services, and optional debugging/metrics export.

## Project Package Structure:

```
cmd/dwh/
├── main.go
```

## Configuration:

*   **Environment Variables:** None explicitly mentioned, but `app.ConfigPath` suggests a configuration file path can be set via environment.
*   **Command-Line Arguments:** Handled by `cmd.NewCmd(run).Execute()`, but specific arguments are not detailed.
*   **Configuration File:** Loaded from `app.ConfigPath` using `dwh.NewDWHConfig`. Contains settings for logging, Ethereum, storage, blockchain, worker count, cold start, metrics, and debugging.
*   **Private Key:** Loaded from the configuration using `cfg.Eth.LoadKey()`.

## Edge Cases:

*   The application can be launched via `cmd.NewCmd(run).Execute()`, but specific command-line flags are not defined.
*   Configuration file errors will likely cause startup failures.
*   Ethereum key loading failures will prevent service initialization.

## Relations:

*   `main.go` orchestrates the entire process: configuration loading, service initialization, and concurrent execution.
*   `dwh.NewDWH` creates the core DWH service, which likely interacts with storage and blockchain components.
*   `dwh.NewL1Processor` handles L1 events, potentially interacting with the DWH service.
*   `logging.BuildLogger` provides structured logging throughout the application.
*   `metrics.NewPrometheusExporter` exports metrics for monitoring.
*   `debug.ServePProf` enables debugging via PProf.