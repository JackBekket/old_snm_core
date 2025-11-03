```markdown
# main - Distributed Warehouse (DWH) Service Summary

This package implements a distributed warehouse (DWH) service with an L1 events processor, metrics exporter, and optional debug server. It's designed to process blockchain data and serve it through a DWH interface. The core functionality revolves around loading configuration, initializing logging, creating the DWH instance, starting the L1 event processing pipeline, serving metrics via Prometheus, and optionally enabling pprof debugging.

## Project Package Structure:

*   `main.go`: Main entry point for the application. Contains `run`, which orchestrates service initialization, execution, and shutdown.

## Configuration & Environment Variables:

The application is configured through a configuration file loaded at runtime. Key configurable parameters include:

*   **`app.ConfigPath`**: Path to the DWH configuration file (e.g., `/etc/dwh/config.yaml`).
*   **`cfg.Eth.LoadKey`**: Private key for Ethereum operations, read from the config.
*   **`cfg.MetricsListenAddr`**: Address where Prometheus metrics are exposed (e.g., `:9090`).
*   **`cfg.Debug.EnablePprof`**: Boolean flag to enable pprof debugging server.

## Launching & Edge Cases:

The application is launched via `go run main.go`. The primary edge case is the failure to load the configuration file or initialize logging, which results in immediate termination.  If the Ethereum private key cannot be loaded from the config, the service will fail to start. If metrics server fails to bind to the configured address, it won't expose any data.

## Code Logic Summary:

The `run` function initializes a DWH service and an L1 event processor based on configuration parameters. It starts concurrent goroutines for signal handling, Prometheus metrics export, optional pprof debugging, and the core processing pipeline (L1 processor). The application shuts down gracefully upon receiving an interrupt signal or encountering an error in any of these goroutines.  The `errgroup` ensures that all resources are cleaned up before exiting.

## Potential Issues:

No explicit TODOs were found within the provided code snippet, but potential issues could arise from misconfigured blockchain settings, storage failures, or network connectivity problems during event processing.