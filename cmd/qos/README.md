# QOS Package Summary

This package implements a gRPC-based Quality of Service (QOS) server with remote GPU tuning and system initialization capabilities, likely part of the SONM ecosystem. It uses configuration loading from YAML files via `configor` to define endpoint, logging settings, GPU vendor, secure shell configurations, and other parameters. The main functionality revolves around exposing gRPC services for QOS management, GPU tuning, and system initialization through a configured endpoint (Unix socket or network address). A background worker monitors a keystore directory for changes and runs a remote PTY server if enabled in the configuration.

**Project Package Structure:**

```
cmd/qos/
├── main.go
```

**Configuration:**

*   **`app.ConfigPath`**: Command-line argument specifying the path to the YAML configuration file (default: `/etc/sonm/qos.yaml`).
*   **YAML Configuration File (`/etc/sonm/qos.yaml`)**: Defines endpoint, logging settings, GPU vendor, system initialization parameters, secure shell configurations, and other runtime options.

**Environment Variables:** None explicitly used in the provided code snippet.

**Command-Line Arguments:**

*   `--config`: Specifies the path to the YAML configuration file (overrides default).
*   Other flags from `github.com/sonm-io/core/cmd` may be present but are not detailed here.

**Edge Cases / Launch Scenarios:**

1.  **Default Configuration**: If no `--config` flag is provided, it loads `/etc/sonm/qos.yaml`.
2.  **Custom Configuration**: Using `--config <path>`, the server reads from a specified YAML file.
3.  **Unix Socket vs Network Address**: The `Endpoint` field in the config determines whether the server listens on a Unix socket or network address (e.g., TCP port).

**Code Relations:**

*   The `main.go` file initializes and runs the gRPC server, loading configuration from YAML using `configor`.
*   Background workers handle QOS requests, GPU tuning, system initialization, and secure shell monitoring.
*   Logging is handled via Zap with context-aware logging (`ctxlog`).