# cmd/worker/main

A minimal bootstrap command for the *worker* component of the **sonm‑io** application.  
The program reads a configuration file, builds a logger and state store, starts a
Prometheus exporter, launches a worker loop and handles graceful shutdown via signals.

---

## Environment variables / flags that influence execution

| Variable / flag | Purpose | Default / source |
|------------------|---------|-------------------|
| `app.ConfigPath` | Path to the YAML/JSON config file read by `worker.NewConfig`. | Provided by the surrounding *cmd* package. |
| `cfg.Logging` | Logging configuration (log level, format, etc.) used when building the logger. | Section of the loaded config. |
| `cfg.Storage` | Storage settings passed to `state.NewState`. | Section of the loaded config. |
| `cfg.MetricsListenAddr` | Address on which the Prometheus exporter will listen. | Section of the loaded config. |

---

## How the application can be launched

* **Development** – `go run ./cmd/worker`
  * Builds and runs the binary directly from source.
* **Production** – `go build -o worker ./cmd/worker && ./worker`
  * Compiles a static binary that can be distributed or containerised.

Both modes honour the same configuration file path (`app.ConfigPath`) and will
start the Prometheus exporter on the address defined in the config.

---

## Project package structure

```
cmd/
└─ worker/
   ├─ main.go          ← this file (package `main`)
```

(Only one source file is present; additional files may exist elsewhere but are not shown.)

---

## Code flow & entity relations

1. **Entry point**  
   ```go
   func main() { cmd.NewCmd(run).Execute() }
   ```
   *Creates a command* that will call the `run` function (defined in this package).

2. **Configuration loading**  
   ```go
   cfg, err := worker.NewConfig(app.ConfigPath)
   ```
   * Reads the config file; `cfg` is later used for logger, state and metrics.

3. **Logger & watcher core**  
   ```go
   watcherCore := logging.NewWatcherCore()
   opts := []zap.Option{ ... }
   logger, err := logging.BuildLogger(cfg.Logging, opts...)
   ctx = log.WithLogger(ctx, logger)
   ```
   * Builds a Zap logger with a custom `WatcherCore` and injects it into the context.

4. **State storage**  
   ```go
   storage, err := state.NewState(ctx, &cfg.Storage)
   ```
   * Creates an in‑memory or persistent state store that will be shared by the worker.

5. **Signal handling goroutine**  
   ```go
   ctx, cancel := context.WithCancel(ctx)
   waiter.Go(func() error {
       c := make(chan os.Signal, 1)
       signal.Notify(c, syscall.SIGINT, syscall.SIGTERM)
       select { ... }
       cancel()
       return nil
   })
   ```
   * Listens for `SIGINT`/`SIGTERM`, logs shutdown and cancels the context.

6. **Worker creation**  
   ```go
   w, err := worker.NewWorker(cfg, storage,
       worker.WithContext(ctx),
       worker.WithVersion(app.Version),
       worker.WithLogWatcher(watcherCore))
   ```
   * Instantiates the core worker logic with all previously built components.

7. **Prometheus exporter**  
   ```go
   go metrics.NewPrometheusExporter(cfg.MetricsListenAddr,
       metrics.WithLogging(logger.Sugar())).Serve(ctx)
   ```
   * Starts an HTTP server that exposes Prometheus metrics on the configured address.

8. **Run & cleanup**  
   ```go
   if err = w.Serve(); err != nil {
       cancel()
       log.G(ctx).Error("server stop", zap.Error(err))
   }
   return waiter.Wait()
   ```
   * Executes the worker’s main loop, logs any error and waits for all goroutines to finish.

---

## Observations

* The code assumes that `run` is defined elsewhere in this package; if it is missing, a compilation error will occur.
* No explicit TODO comments are present – the logic appears complete.
* All external dependencies (`logging`, `state`, `worker`, `metrics`) are used exactly once, indicating tight coupling between configuration, state and metrics.

---