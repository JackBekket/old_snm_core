# Package `logging`

The *logging* package supplies a small, self‑contained logging helper built on top of Uber’s **zap** logger.  
It offers:

1. A YAML‑driven configuration type (`Config`) that stores the desired log level and output stream.  
2. A constructor (`BuildLogger`) that turns a `Config` into a fully configured `*zap.Logger`.  
3. A lightweight wrapper around zap’s `Level` type to ease marshaling/unmarshaling from YAML.  
4. OpenTracing integration (`WithTrace`) that injects trace/span IDs into log entries.  
5. A custom *watcher* core (`WatcherCore`) that can broadcast log messages to multiple observers via a shared channel map.

---

## Environment variables, flags & command‑line arguments

| Source | What it configures | Typical usage |
|--------|--------------------|----------------|
| `LOGGING_CONFIG` (env) | Path to a YAML file containing a `logging.Config`. | `-config=${LOGGING_CONFIG}` or read from default path in code. |
| `--output` (flag) | Desired output stream (`stdout`/`stderr`). | `-output=stdout` – used by `Config.Output`. |
| `--level` (flag) | Log level string (`debug`, `info`, …). | `-level=info` – parsed into `Config.Level`. |

> **Note**: The package itself does not expose a CLI flag parser; it expects the caller to read these values and populate a `logging.Config`.

---

## File structure

```
insonmnia/
└─ logging/
   ├─ config.go          # Config struct, constants, helper fdFromString
   ├─ logging.go         # BuildLogger, Level wrapper, parseLogLevel
   ├─ logging_test.go    # Unit test for parseLogLevel
   ├─ trace.go           # WithTrace helper
   └─ watcher.go         # WatcherCore implementation
```

---

## How the pieces fit together

| File | Key entities | Interaction |
|------|---------------|-------------|
| `config.go` | `Config`, constants, `fdFromString()` | Provides data for `BuildLogger`. |
| `logging.go` | `BuildLogger()`, `Level` type & helpers | Creates a zap logger from a `Config`; exposes `LogLevel()` and `Zap()` methods used by the constructor. |
| `trace.go` | `WithTrace(ctx, log)` | Adds OpenTracing span data to an existing logger; can be chained after `BuildLogger`. |
| `watcher.go` | `WatcherCore`, `newWatcher()`, `Notify()`, `Subscribe()/Unsubscribe()` | Implements a zap core that broadcasts entries to all registered observers. |

The flow for a typical application is:

1. Load a YAML file into a `logging.Config` (e.g., via Viper or another config loader).  
2. Call `logging.BuildLogger(cfg, options...)` to obtain a `*zap.Logger`.  
3. Optionally wrap the logger with tracing: `logger = logging.WithTrace(ctx, logger)`.  
4. If multiple goroutines need to observe log output, create a `WatcherCore`:  
   ```go
   core := logging.NewWatcherCore()
   core.Subscribe() // returns an ID you can later use to unsubscribe
   ```
5. Pass the core into zap’s `zap.New(core)` or use it directly if you only want the custom core.

---

## Edge cases for launching

| Scenario | What to consider |
|----------|------------------|
| **No config file** | The package falls back on defaults (`Level: info`, `Output: stdout`). Ensure a default YAML path is supplied. |
| **Invalid output string** | `fdFromString()` returns `nil`; callers should handle this gracefully (e.g., log an error). |
| **Multiple observers** | `WatcherCore` keeps a thread‑safe map; calling `Subscribe()` multiple times yields distinct IDs that can be used to unsubscribe later. |
| **Tracing context missing** | `WithTrace` silently returns the original logger if no span is found in the context – useful for non‑traced runs. |

---

## Summary of logic

* `config.go` defines a minimal configuration type and helper functions.  
* `logging.go` builds a zap logger from that config, exposing convenient methods to convert between YAML values and zap’s internal level type.  
* `trace.go` enriches the logger with OpenTracing span IDs so logs carry trace information automatically.  
* `watcher.go` implements a custom zap core that can broadcast log entries to multiple observers; it is used by callers who need to observe or forward logs elsewhere.

All files together provide a small, reusable logging stack that can be dropped into any Go project requiring structured logging with optional tracing and observer support.