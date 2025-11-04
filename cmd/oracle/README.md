# oracle

## Overview  
`cmd/oracle/main.go` is the entry point of a small Go command‑line tool that loads configuration, builds a logger and starts an *Oracle* service.  The program then runs two concurrent tasks – a graceful shutdown waiter and the Oracle’s main loop – using `golang.org/x/sync/errgroup`.

---

## Project structure  

```
cmd/oracle/
├── main.go
```

The package name is inferred from the directory (`oracle`) and the file contents; it is used as the command‑line binary.

---

## Environment variables, flags & configuration files

| Variable / File | Purpose |
|------------------|---------|
| `app.ConfigPath` | Path to a YAML/JSON config that `oracle.NewConfig` reads. |
| `cfg.Log` | Logging settings passed to `logging.BuildLogger`. |
| `cmd.WaitInterrupted(ctx)` | Waits for an interrupt signal (e.g., SIGINT). |
| `o.Serve(ctx)` | Starts the Oracle service loop. |

No command‑line flags are defined in this file; all configuration is read from the config path.

---

## How to launch

* **Development** – `go run ./cmd/oracle`  
  Executes the binary directly, using the current working directory as the root of the module.
* **Production** – `go build -o oracle ./cmd/oracle && ./oracle`  
  Builds a standalone executable named *oracle* that can be invoked from any location.

Both approaches rely on the same environment variable (`app.ConfigPath`) and logger configuration.

---

## Code walk‑through

### 1. `main()`  

```go
func main() {
    cmd.NewCmd(run).Execute()
}
```

Creates a new command object with the helper from `github.com/sonm-io/core/cmd` and immediately executes it, delegating all runtime logic to `run`.

### 2. `run(app cmd.AppContext) error`  

```go
func run(app cmd.AppContext) error {
    cfg, err := oracle.NewConfig(app.ConfigPath)
    if err != nil { … }

    logger, err := logging.BuildLogger(cfg.Log)
    if err != nil { … }

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    o, err := oracle.NewOracle(ctx, logger, cfg)
    if err != nil { … }

    wg, ctx := errgroup.WithContext(ctx)
    …
}
```

* Loads configuration (`oracle.NewConfig`) from the path supplied by `app.ConfigPath`.  
* Builds a logger with those settings.  
* Creates a cancellable context for the whole run.  
* Instantiates an Oracle instance (`o`) that will serve requests.

### 3. Goroutine orchestration  

```go
wg.Go(func() error {
    return cmd.WaitInterrupted(ctx)
})

wg.Go(func() error {
    return o.Serve(ctx)
})
```

Two concurrent workers are launched:

1. `cmd.WaitInterrupted` – likely blocks until an interrupt signal is received, then cancels the context.
2. `o.Serve` – runs the Oracle’s main loop.

Both run in parallel; any returned error will be captured by the group.

### 4. Completion  

```go
if err := wg.Wait(); err != nil {
    return fmt.Errorf("termination: %s", err)
}
```

Waits for both goroutines to finish and propagates any error that occurs.  
On success, `run` returns `nil`.

---

## Relations & potential dead code

* The only external dependency is the Oracle package (`github.com/sonm-io/core/insonmnia/oracle`).  
* No obvious dead code or missing references are present; all variables are used.
* The context variable shadowing in `wg, ctx := errgroup.WithContext(ctx)` is intentional – it extends the original context with cancellation support.

---