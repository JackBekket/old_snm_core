# Package `node` – Application Bootstrap

The file **cmd/node/main.go** contains the entry point for a command‑line executable that loads configuration, creates a logger, starts a node server and exposes Prometheus metrics.  
It is the glue code that turns the various sub‑packages (`logging`, `node`, `metrics`) into one runnable binary.

---

## Short Summary

* Loads a config file via `node.NewConfig(app.ConfigPath)`.
* Builds a structured logger with `logging.BuildLogger(cfg.Log)`.
* Wraps a background context with the logger and validates the application version.
* Starts three concurrent goroutines:
  1. Waits for an interrupt signal (`cmd.WaitInterrupted`).
  2. Creates a node instance (`node.New`) and serves it.
  3. Runs a Prometheus metrics exporter on `cfg.MetricsListenAddr`.
* All errors are propagated back to the caller; the binary exits when all goroutines finish.

---

## Environment Variables, Flags & Command‑Line Arguments

| Variable / Flag | Purpose |
|------------------|---------|
| `app.ConfigPath` (flag) | Path to a YAML/JSON config file that contains logging and metrics settings. |
| `cfg.MetricsListenAddr` | Address on which the Prometheus exporter listens. |
| `cfg.Log` | Logger configuration section used by `logging.BuildLogger`. |

---

## Project Package Structure

```
cmd/
└─ node/
   └─ main.go
```

*Only one source file is present; it defines the binary entry point.*

---

## Relations Between Code Entities

1. **`main()`** – creates a command (`cmd.NewCmd(run)`) and immediately executes it.  
2. **`run(ctx context.Context, app *App)`** (defined in the same file) – performs all initialization steps described above.  
3. **`node.NewConfig(app.ConfigPath)`** – reads configuration from disk; its return value is stored in `cfg`.  
4. **`logging.BuildLogger(cfg.Log)`** – creates a logger that is attached to the context via `ctxlog.WithLogger`.  
5. **`errgroup.WithContext(ctx)`** – provides a wait group that runs three goroutines concurrently: interrupt handling, node serving, and metrics exporting.

---

## Edge Cases & Launch Options

| Scenario | Command |
|----------|---------|
| Run directly from source | `go run ./cmd/node` |
| Build binary for distribution | `go build -o bin/node ./cmd/node && ./bin/node --config=...` |

The binary can be started with the usual Go flags (`--config`, etc.) that populate `app.ConfigPath`. Once launched, it will keep running until an interrupt signal is received or all goroutines finish.

---

## Summary of Logic

1. **Bootstrap** – `main()` starts the command.
2. **Configuration** – `run` loads config and logger.
3. **Context & Version** – context enriched with logger; version validated.
4. **Concurrent Tasks** – three goroutines: interrupt wait, node serve, metrics export.
5. **Completion** – waits for all tasks to finish; propagates any error.

This file is the single orchestrator that turns configuration and node logic into a working command‑line application.