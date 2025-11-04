# Package: `debug`

This package provides a simple HTTP server that exposes pprof endpoints for profiling Go applications. It allows monitoring CPU, memory, and goroutine usage.

## Configuration

*   **Environment Variables:** None explicitly used, but the port is configurable via YAML.
*   **Flags/Cmdline Arguments:** None.
*   **Files:** `util/debug/pprof.go`
*   **Paths:** `util/debug`

## Launch Edgecases

The `ServePProf` function takes a `context.Context` as input. If the context is cancelled, the server shuts down. The port is configurable via YAML. If no port is specified, it defaults to 6060.

## Package Structure

```
util/
└── debug/
    └── pprof.go
```

## Code Relations

The `ServePProf` function uses the `newHandler` function to create the HTTP handler that exposes pprof endpoints. The `Config` struct defines the port configuration. The `zap.Logger` is used for logging server start and stop events.