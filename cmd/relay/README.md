# Relay Server Package

## Overview
`cmd/relay/main.go` implements the entry point for a **Relay‑based server** application.  
It loads configuration, builds a logger, creates a `relay.Server`, and runs it concurrently with graceful shutdown handling.

---

## Project package structure

```
cmd/
└─ relay/
   ├─ main.go
```

*Only one source file is present in this package.*

---

## Imports & External Dependencies

| Package | Alias |
|---------|-------|
| `context` | – |
| `fmt` | – |
| `github.com/noxiouz/zapctx/ctxlog` | `log` |
| `github.com/sonm-io/core/cmd` | – |
| `github.com/sonm-io/core/insonmnia/logging` | – |
| `github.com/sonm-io/core/insonmnia/npp/relay` | – |
| `golang.org/x/sync/errgroup` | – |

---

## Configuration sources

| Source | Path / Variable | Description |
|--------|-----------------|-------------|
| `app.ConfigPath` | `cmd.AppContext.ConfigPath` | File path to the Relay server configuration (YAML/TOML/etc.). |
| `cfg.Logging` | `relay.NewServerConfig(...).Logging` | Logging settings passed to `logging.BuildLogger`. |
| `*cfg` | Dereferenced config struct | Full configuration forwarded to `relay.NewServer`. |

---

## Environment variables, flags & command‑line arguments

The package itself does not expose any custom environment variables or CLI flags; it relies on the surrounding `cmd.AppContext` infrastructure.  
Typical usage:

```bash
# Build and run
go build ./cmd/relay
./relay --config /path/to/config.yaml
```

If the surrounding framework supports flag parsing, the following are expected:

| Flag | Description |
|------|-------------|
| `--config` (or similar) | Path to the configuration file. |

---

## Code flow

### `start(app cmd.AppContext) error`
1. **Load config** – `relay.NewServerConfig(app.ConfigPath)`  
   * Returns a `cfg` struct; errors are wrapped with context.
2. **Build logger** – `logging.BuildLogger(cfg.Logging)`  
3. **Prepare context** – `log.WithLogger(context.Background(), log.G(ctx))` and build options slice (`relay.WithLogger(log.G(ctx))`).  
4. **Instantiate server** – `relay.NewServer(*cfg, options...)`.  
5. **Run concurrently** – an `errgroup` is created; two goroutines are launched:  
   * `server.Serve(ctx)` – main server loop.  
   * `cmd.WaitInterrupted(ctx)` – waits for interrupt signal and triggers shutdown.  
6. **Return** – function returns `nil` on success.

### `main()`
Creates a new command via `cmd.NewCmd(start)` and executes it, wiring the `start` routine into the CLI handling logic of the application.

---

## Edge cases & launch scenarios

| Scenario | How to launch |
|----------|---------------|
| **Development** | `go run ./cmd/relay` – runs directly from source. |
| **Production binary** | `go build -o relay ./cmd/relay && ./relay` – builds a standalone executable. |
| **With custom config path** | Pass the config file via an environment variable or CLI flag that populates `app.ConfigPath`. |

---

## Summary

The package provides a minimal yet complete bootstrap for a Relay server: it reads configuration, sets up logging, creates the server instance, and runs it with graceful shutdown. All logic is encapsulated in two functions (`start` and `main`) and relies on external packages for context handling, logging, and error grouping.