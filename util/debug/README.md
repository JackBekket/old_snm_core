# Package `debug`

## Overview  
`util/debug/pprof.go` implements a lightweight HTTP server that exposes Go’s built‑in profiler (`net/http/pprof`) on a configurable port. The server listens on `localhost:<port>` and registers all standard pprof endpoints under the prefix `/debug/pprof`. It is intended to be used by other components of the project for runtime diagnostics.

---

## Environment variables, flags & command‑line arguments  
| Source | Key / Flag | Description |
|--------|------------|-------------|
| YAML config file (e.g. `config.yaml`) | `port` | Port number on which the server will listen (`default: 6060`). |
| Context passed to `ServePProf` | – | Allows graceful shutdown via cancellation. |
| Logger passed to `ServePProf` | – | Uses `go.uber.org/zap.Sugar()` for logging. |

> **Note** – No explicit command‑line flags are defined in this file; configuration is expected to come from a YAML source.

---

## Project package structure  

```
util/
└─ debug/
   └─ pprof.go
```

* `pprof.go` – the sole source file of the `debug` component.

---

## Code summary

| Section | Purpose |
|---------|---------|
| **Imports** | Bring in context handling, formatting, networking, HTTP server primitives, Go’s profiler package, and Zap logging. |
| **Constants** | `prefix = "/debug/pprof"` – URL prefix for all handlers; `ipAddr = "localhost"` – IP address to bind to. |
| **Config struct** | Holds the listening port (`yaml:"port" default:"6060"`). |
| **`ServePProf(ctx, cfg, logger)`** | 1. Builds a TCP address string from config.<br>2. Creates a listener with `net.Listen`. <br>3. Starts an HTTP server in a goroutine that serves all profiling endpoints via `newHandler(prefix)`. <br>4. Logs start/stop messages and blocks until the context is cancelled. |
| **`newHandler(prefix)`** | Constructs an `http.ServeMux`, registers handlers for `pprof.Index`, `Cmdline`, `Profile`, `Symbol`, and `Trace` under the given prefix, then returns it. |

---

## Relations between code entities

* `ServePProf` depends on `Config.Port`; it passes that value to `newHandler`.
* The handler returned by `newHandler` is used as the server’s root handler in the goroutine started inside `ServePProf`.
* Logging uses the passed‑in `zap.Logger`, so any component calling `ServePProf` can inject its own logger.

---

## Edge cases & launch scenarios

| Scenario | How to run |
|----------|------------|
| **Standalone debugging server** – If this package is compiled as a binary (e.g. via `go build ./util/debug`), the resulting executable will start an HTTP server on `localhost:6060`. |
| **Integrated into a larger application** – Other packages can import `"util/debug"` and call `ServePProf(ctx, cfg, logger)` to add profiling endpoints to their own HTTP service. |

> **Tip** – To change the listening port, edit the YAML config file or pass a different value for `Config.Port` when calling `ServePProf`.

---