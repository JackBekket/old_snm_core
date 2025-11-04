## Package: `debug`

This package provides functionality to start a pprof HTTP server for debugging and performance analysis. It serves standard pprof endpoints (index, cmdline, profile, symbol, trace) on a configurable port (default 6060). The server runs in a goroutine managed by a context.

**Configuration:**

*   Port number is read from YAML configuration; defaults to 6060 if not specified.

**Files:**

*   `util/debug/pprof.go`: Contains the `ServePProf` function (starts the server) and `newHandler` (creates the HTTP handler).

**Functions:**

*   `ServePProf(ctx context.Context, config Config)`: Starts a pprof server on the configured port.
*   `newHandler()`: Creates an HTTP handler for serving pprof endpoints under "/debug/pprof".

**Usage:**

The package is intended to be used as part of a larger application where debugging and profiling are required. The `ServePProf` function should be called within the main application's startup sequence, passing in a context that can be used to shut down the server gracefully.  Configuration (port number) is expected via YAML or similar external source.