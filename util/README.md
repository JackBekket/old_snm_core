Okay, here's a markdown summary of the provided package code, following your instructions.

# util Package - Core Utility Functions

**Package Name:** `util`

**Summary:**

The `util` package provides a collection of core utility functions for various tasks, including certificate handling, configuration management, gRPC integration, and general system utilities. It includes functions for file system operations, Ethereum-related data manipulation, and custom TLS certificate rotation. The package appears to be designed for a distributed system or blockchain-related application, given the Ethereum-specific functions and TLS certificate rotation logic.

**Configuration:**

*   **Environment Variables:** None explicitly defined in the provided code.
*   **Flags/Cmdline Arguments:** None explicitly defined in the provided code.
*   **Files/Paths:**
    *   Configuration directory: `~/.sonm` (default)
    *   Keystore directory: `~/.sonm/keystore` (default)
*   **Edge Cases (Launch):** Not applicable, as this is a utility package, not a standalone application.

**Project Package Structure:**

```
util/
├── action/
│   ├── action.go
│   └── queue.go
├── certs.go
├── certs_test.go
├── config/
│   ├── config.go
│   ├── retag.go
│   └── retag_test.go
├── datasize/
│   ├── datasize.go
│   └── datasize_test.go
├── debug/
│   └── pprof.go
├── defergroup/
│   └── mod.go
├── grpc.go
├── grpcsecure.go
├── grpcutil_test.go
├── metadata.go
├── metrics/
│   └── prometheus.go
├── multierror/
│   └── error.go
├── netutil/
│   ├── net.go
│   └── net_test.go
├── rest/
│   ├── aes.go
│   ├── errors.go
│   ├── options.go
│   └── server.go
├── ticker.go
├── util.go
├── util_test.go
├── xcode/
│   └── cmd.go
├── xconcurrency/
│   └── cncurrency.go
├── xdocker/
│   ├── reference.go
│   ├── reference_test.go
│   ├── xdocker.go
│   └── xdocker_test.go
├── xgrpc/
│   ├── client.go
│   ├── client_test.go
│   ├── credentials.go
│   ├── method.go
│   ├── metrics.go
│   ├── options.go
│   ├── server.go
├── xnet/
│   ├── listener.go
│   ├── quic.go
│   └── resolve.go
```

**Relations Between Code Entities:**

*   **`certs.go` and `grpcsecure.go`:** The certificate rotation logic in `certs.go` is likely used to secure gRPC connections in `grpcsecure.go`.
*   **`util.go` and other files:** The core utility functions in `util.go` (file system operations, number conversions) are likely used by other modules within the package.
*   **`xgrpc/*`:** This directory contains gRPC-specific utilities, such as client and server implementations, credentials management, and metrics integration.
*   **`xnet/*`:** This directory contains network-related utilities, including listener management and QUIC support.
*   **`config/*`:** This directory contains configuration-related utilities, including retagging functionality.

**Unclear Places/Dead Code:**

*   The exact purpose of some files (e.g., `action/`, `defergroup/`, `xcode/`) is unclear without further context.
*   The `xconcurrency/` directory appears to contain concurrency-related utilities, but its specific use cases are not evident.
*   The `multierror/` directory suggests error handling utilities, but the implementation details are not provided.
*   The `rest/` directory suggests REST-related utilities, but the implementation details are not provided.
*   The `metrics/prometheus.go` suggests integration with Prometheus for monitoring, but the implementation details are not provided.

The package appears to be a comprehensive set of utilities for a complex system, but the lack of high-level documentation makes it difficult to fully understand its purpose and dependencies.