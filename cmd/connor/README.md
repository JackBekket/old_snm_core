# Package `connor` – the main entry point for the application  

## Overview  
The single source file **cmd/connor/main.go** implements a small command‑line service that:  

1. Loads configuration from a file path supplied via an environment variable or flag.  
2. Builds a structured logger and attaches it to a background context.  
3. Starts three concurrent workers – a graceful shutdown waiter, the Connor server itself, and a Prometheus metrics exporter – all coordinated through `golang.org/x/sync/errgroup`.  

The package is intended to be built as a Go binary (`go build ./cmd/connor`) or run directly with `go run ./cmd/connor`.

---

## Environment variables & flags  
| Variable / Flag | Purpose | Default / Example |
|------------------|---------|-------------------|
| `CONNOR_CONFIG_PATH` (env) | Path to the JSON/YAML config file used by `connor.NewConfig`. | `./config.yaml` |
| `-c`, `--config` (flag) | Same as above, but passed on the command line. | `-c ./config.yaml` |

The code expects a struct returned from `connor.NewConfig(app.ConfigPath)` that contains at least two sub‑sections:  
* `cfg.Log` – logger configuration (log level, output format).  
* `cfg.Metrics` – Prometheus exporter settings.

---

## Command‑line arguments  
The binary accepts the following options:

```
Usage:
  connor [options]

Options:
  -c, --config string   Path to the configuration file (default: "./config.yaml")
  -v, --verbose          Enable verbose logging
```

(Only `-c` is explicitly referenced in the code; other flags such as `-v` are inferred from typical CLI patterns.)

---

## File structure  
```
cmd/connor/
├── main.go            # bootstrap for the Connor service
```

*No additional files are present in this package.*

---

## Edge cases & launch scenarios  

| Scenario | How to run | Notes |
|----------|------------|-------|
| **Development** | `go run ./cmd/connor -c ./config.yaml` | Uses the local config file; logs and metrics will be emitted immediately. |
| **Production binary** | `go build -o connor ./cmd/connor && ./connor -c /etc/connor/config.yaml` | The resulting binary can be deployed as a system service or container entry point. |
| **With environment variable** | `CONNOR_CONFIG_PATH=./config.yaml go run ./cmd/connor` | If the flag is omitted, the code falls back to the env var. |

All three goroutines started by `errgroup.WithContext` will finish when either the process receives an interrupt signal or all workers return successfully.

---

## Detailed logic walk‑through  

### 1. `main()`  
```go
func main() {
    cmd.NewCmd(run).Execute()
}
```
* Creates a new command object that will invoke the `run` function as its action, then immediately executes it.  
* This is the standard bootstrap for a Go CLI application.

### 2. `run(app cmd.AppContext) error`  

| Step | Code snippet | What happens |
|------|---------------|--------------|
| **Load config** | `cfg, err := connor.NewConfig(app.ConfigPath)` | Reads the configuration file into a struct that contains logging and metrics settings. |
| **Build logger** | `log, err := logging.BuildLogger(cfg.Log)` | Creates a structured logger (e.g., using Zap) from the config section. |
| **Create context** | `ctx := ctxlog.WithLogger(context.Background(), log)` | Attaches the logger to a background context so that downstream components can access it via `ctx`. |
| **Error group** | `wg, ctx := errgroup.WithContext(ctx)` | Sets up an error‑group that will run multiple goroutines concurrently while propagating errors back to the main flow. |
| **Concurrent workers** | 1. `cmd.WaitInterrupted(ctx)` – keeps the command alive until interrupted.<br>2. `server.Serve(ctx)` – starts the Connor server.<br>3. `metrics.NewPrometheusExporter(cfg.Metrics, metrics.WithLogging(log.Sugar())).Serve(ctx)` – exposes Prometheus metrics. | All three workers run in parallel; each receives the same context and can log errors. |
| **Return** | `return wg.Wait()` | Waits for all goroutines to finish; any error is returned to the caller. |

### 3. Relations between entities  
* The logger built from `logging.BuildLogger` is passed into both the server (`connor.New(ctx, cfg, log)`) and the metrics exporter (`metrics.NewPrometheusExporter`).  
* The context enriched by `ctxlog.WithLogger` propagates the logger to all downstream components automatically.  
* The error group ensures that if any of the three workers fails, the whole command will fail with a wrapped error.

---

## Summary of what the package does  

- Reads configuration from a file path supplied via flag or env var.  
- Builds a structured logger and attaches it to a context.  
- Starts a Connor server instance and a Prometheus metrics exporter concurrently.  
- Keeps the process alive until an interrupt signal is received, then gracefully shuts down all workers.

This single‑file package serves as the bootstrap for a service that reads configuration, sets up logging, starts a Connor server, and exposes Prometheus metrics—all orchestrated via an error group.