# metrics

A lightweight system‑metrics collector for the *insonmnia* worker stack.  
The package exposes a single `Handler` type that aggregates CPU, GPU, disk and RAM statistics into a protobuf message (`sonm.WorkerMetricsResponse`).  The handler can be started as a background goroutine that updates every minute, making it suitable for use in a long‑running service or as part of a CLI tool.

---

## Environment variables / flags / command‑line arguments

| Variable / flag | Purpose | Default |
|------------------|---------|---------|
| `METRIX_LOG_LEVEL` | Log level for the handler (used by `zap`) | `info` |
| `METRIX_TICKER_INTERVAL` | Update interval in seconds | `60` |
| `METRIX_GPU_CONFIG` | JSON/YAML file path containing GPU vendor → config map | – |

> **Note**: The package itself does not parse any command‑line arguments; it expects a logger and a GPU configuration map to be passed programmatically.  If used from a CLI, the caller should read `METRIX_GPU_CONFIG` into a `map[string]map[string]string` and hand it to `NewHandler`.

---

## Project package structure

```
insonmnia/
├─ worker/
│   ├─ metrics/
│   │   └─ metrics.go
└─ (other packages)
```

*Only one source file is present in this sub‑package.*

---

## Core code flow

| Function | Responsibility |
|----------|----------------|
| `NewHandler` | Builds a new `Handler`, creates GPU metric handlers from the supplied config, and stores them in a map keyed by vendor type. |
| `Run` | Starts an infinite ticker that calls `update` every minute; stops when the context is cancelled. |
| `update` | Orchestrates all sub‑updates (GPU, CPU, disk, RAM), aggregates errors with `multierror`, and writes the resulting protobuf into `lastState`. |
| `updateGPUMetrics` | Iterates over each GPU handler, collects its metrics via `GetMetrics`, and merges them into a single map. |
| `updateCPUMetrics` | Calls `cpu.Percent` to get CPU utilisation; returns a map with key `sonm.MetricsKeyCPUUtilization`. |
| `updateDiskMetrics` | Wraps the call to `disk.FreeDiskSpace`; builds a map of total and free bytes. |
| `updateRAMMetrics` | Creates a RAM device via `ram.NewRAMDevice`; returns a map containing total and free RAM values. |
| `Get` | Thread‑safe accessor for the latest metrics snapshot. |
| `Close` | Gracefully shuts down all GPU handlers and aggregates any errors. |

The handler’s internal state (`lastState`) is protected by a mutex, so concurrent reads/writes are safe.

---

## Edge cases / launch scenarios

* **As a library** – Import `insonmnia/worker/metrics`, create a logger with `zap.New(...)`, read GPU config into a map, call `NewHandler`, then `Run(ctx)` in a background goroutine.  
* **As part of a CLI** – A small wrapper program can parse command‑line flags (`--gpu-config`, `--interval`) and invoke the same sequence; the handler will automatically tick every minute until the process exits or receives a SIGTERM.  

---

## Summary

The package implements a single, thread‑safe collector that periodically gathers CPU, GPU, disk, and RAM metrics into a protobuf message.  The only public API is `Handler`, which can be instantiated with a logger and a GPU configuration map, started with `Run`, read with `Get`, and stopped with `Close`.  All sub‑updates are isolated in dedicated helper methods for clarity and future extension.