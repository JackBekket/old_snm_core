# Package `rv`

The **`cmd/rv/main.go`** file is the entry point of a small command‑line application that starts a *Rendezvous* server.  
It pulls configuration from an external file, builds a logger and TLS cert rotator, creates a server instance with QUIC support, then runs it in parallel with a graceful shutdown helper.

---

## 1. File structure

```
cmd/rv/
├── main.go          ← executable entry point
```

The package name is inferred from the directory (`rv`).

---

## 2. Imports & purpose

| Import | Purpose |
|--------|---------|
| `context` | Standard Go context handling |
| `fmt` | Error formatting and logging |
| `log "github.com/noxiouz/zapctx/ctxlog"` | Context‑aware logger wrapper |
| `github.com/sonm-io/core/cmd` | CLI helper (flag parsing, command registration) |
| `github.com/sonm-io/core/insonmnia/logging` | Logger construction helper |
| `github.com/sonm-io/core/insonmnia/npp/rendezvous` | Rendezvous server configuration & runtime |
| `github.com/sonm-io/core/util` | Utility helpers (e.g. cert rotator) |
| `golang.org/x/sync/errgroup` | Parallel goroutine execution with error handling |

---

## 3. Environment variables, flags and command‑line arguments

| Variable / Flag | Default / Example | Description |
|------------------|-------------------|-------------|
| `RVD_CONFIG_PATH` (env) | `./config.yaml` | Path to the server configuration file |
| `-config` (flag) | same as above | Override config path at runtime |
| `-key` (flag) | path to a private key file | TLS cert rotator source |

The code expects these values either from environment or CLI flags; they are passed into `rendezvous.NewServerConfig`.

---

## 4. Core logic

1. **Load configuration**  
   ```go
   cfg, err := rendezvous.NewServerConfig(app.ConfigPath)
   ```
   Reads the YAML/JSON config file and returns a struct used throughout.

2. **Build logger**  
   ```go
   logger, err := logging.BuildLogger(cfg.Logging)
   ```

3. **Create cert rotator**  
   ```go
   certRotator, TLSConfig, err := util.NewHitlessCertRotator(ctx, cfg.PrivateKey)
   ```
   The rotator is deferred for cleanup.

4. **Prepare server options**  
   ```go
   opts := []rendezvous.Option{
       rendezvous.WithCredentials(TLSConfig),
       rendezvous.WithQUIC(),
       rendezvous.WithLogger(log.G(ctx)),
   }
   ```

5. **Instantiate server**  
   ```go
   server, err := rendezvous.NewServer(cfg, opts...)
   ```

6. **Run in parallel**  
   ```go
   wg, ctx := errgroup.WithContext(ctx)
   wg.Go(func() error { return server.Run(ctx) })
   wg.Go(func() error { return cmd.WaitInterrupted(ctx) })
   ```

7. **Wait for completion and log**  
   ```go
   if err := wg.Wait(); err != nil {
       log.S(ctx).Infof("rendezvous server is stopped: %v", err)
   }
   ```

8. **Entry point**  
   ```go
   func main() { cmd.NewCmd(start).Execute() }
   ```
   Registers the `start` function as a command handler and executes it.

---

## 5. Edge cases & launch scenarios

| Scenario | How to launch |
|----------|---------------|
| Local dev | `go run ./cmd/rv/main.go -config=./dev.yaml` |
| Production build | `go build -o rv ./cmd/rv` then `./rv -config=./prod.yaml` |
| With env var | `RVD_CONFIG_PATH=./prod.yaml go run ./cmd/rv/main.go` |

The program will log startup, run the server until interrupted (e.g. Ctrl‑C), and exit cleanly.

---

## 6. Summary

* The file defines a single command (`start`) that loads configuration, builds a logger, sets up TLS cert rotation, creates a QUIC‑enabled Rendezvous server, runs it concurrently with graceful shutdown handling, and logs completion.
* All configuration values are read from the config file; no hard‑coded paths or flags beyond those mentioned above.

---