# dwh

## Overview  
`cmd/dwh/main.go` is the bootstrap for a **Data‑Warehouse (DWH)** command line tool that orchestrates an L1 event processor and exposes Prometheus metrics.  
The binary loads its configuration, builds a logger, starts two services (`dwh.NewDWH` & `dwh.NewL1Processor`) and runs them concurrently with graceful shutdown handling.

---

## Environment / Configuration

| Source | Key | Description |
|--------|-----|-------------|
| **Config file** | `app.ConfigPath` | Path to a YAML/JSON config that contains: <br>`Logging`, `Eth`, `MetricsListenAddr`, `Debug`. |
| **Ethereum key** | `cfg.Eth.LoadKey()` | Private key used by the DWH service. |
| **Processor settings** | `&dwh.L1ProcessorConfig{Storage, Blockchain, NumWorkers, ColdStart}` | Sub‑config for the L1 processor. |
| **Metrics address** | `cfg.MetricsListenAddr` | Address where Prometheus metrics are served. |
| **Debug server** | optional `*cfg.Debug` | If present, a pprof debug server is started. |

---

## Command line / Flags

The binary is invoked as:

```bash
dwh [--config <path>] [--metrics-addr <addr>]
```

All options are parsed by the `cmd.NewCmd(run)` helper; the only required flag is the config path (`app.ConfigPath`).  
No other explicit flags appear in this file.

---

## File structure

```
cmd/dwh/
├── main.go
```

Only one source file is present, but it imports several packages that provide the actual logic (see below).

---

## Code walk‑through

### 1. `main()`

```go
func main() {
    cmd.NewCmd(run).Execute()
}
```

* Builds a command that will execute the `run` function and immediately runs it.

### 2. `run(app cmd.AppContext) error`

The orchestration routine:

| Step | Action |
|------|--------|
| **Load config** | `cfg, err := dwh.NewDWHConfig(app.ConfigPath)` |
| **Build logger** | `logger, err := logging.BuildLogger(cfg.Logging)` |
| **Create context** | `ctx := log.WithLogger(context.Background(), logger)` |
| **Log start** | `logger.Info("starting with config", zap.Any("config", cfg))` |
| **Load Ethereum key** | `key, err := cfg.Eth.LoadKey()` |
| **Instantiate services** | `w, err := dwh.NewDWH(ctx, cfg, key)`<br>`p, err := dwh.NewL1Processor(ctx, &dwh.L1ProcessorConfig{…})` |
| **Concurrent workers** | `wg, ctx := errgroup.WithContext(ctx)` and five goroutines: <br>1. graceful shutdown (`cmd.WaitInterrupted`) <br>2. Prometheus exporter (`metrics.NewPrometheusExporter`) <br>3. optional debug pprof server (`debug.ServePProf`) <br>4. start L1 processor (`p.Start()`) <br>5. serve DWH service (`w.Serve()`). |
| **Wait** | `return wg.Wait()` |

---

## Edge cases / launch scenarios

* **Normal run** – simply execute the binary; it will read the config, start services and expose metrics.
* **With debug server** – if a `Debug` section exists in the config, the third goroutine starts an HTTP pprof server.
* **Graceful shutdown** – pressing Ctrl‑C triggers `cmd.WaitInterrupted`, which stops both processor and DWH service in order.

---

## Summary

The file defines a lightweight CLI entry point that:

1. Loads configuration from a path supplied by the command line,
2. Builds a Zap logger,
3. Creates an execution context,
4. Starts two services (DWH core & L1 processor) concurrently,
5. Exposes Prometheus metrics and optional debug pprof, and
6. Handles graceful shutdown.

All heavy logic lives in the imported packages (`dwh`, `logging`, `metrics`, etc.), while this file wires them together.