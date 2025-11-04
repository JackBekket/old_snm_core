# Package qos

A lightweight command‑line entry point for a QOS (Quality of Service) service that wires together configuration loading, logging, GPU tuning and optional PTY handling over a Unix socket.

---

## Quick Overview
`cmd/qos/main.go` loads a YAML config file into a `Config` struct, builds a logger, starts a gRPC server on a Unix domain socket, registers three services (`remoteQOS`, `remoteTuner`, `remoteInit`) and optionally launches a PTY server if a secsh configuration is present. All heavy lifting is performed concurrently via an `errgroup.Group`.

---

## Environment Variables / Flags / Cmd‑line Arguments

| Variable / Flag | Description | Default |
|------------------|-------------|---------|
| `app.ConfigPath` | Path to the YAML config file that will be unmarshaled into `Config`. | – |
| `cfg.Logging` | Logger configuration passed to `logging.BuildLogger`. | – |
| `cfg.GPUVendor` | GPU vendor string used by `gpu.NewRemoteTuner`. | – |
| `cfg.SysInit` | System‑initialization config for `sysinit.NewInitService`. | – |
| `cfg.SecShell` | Optional secsh configuration; if present a PTY server is started. | – |
| `cfg.Endpoint` | URI (default: `unix:///var/run/qos.sock`) used to create the network listener and expose services. | `unix:///var/run/qos.sock` |

---

## Project Package Structure

```
cmd/
└─ qos/
   ├─ main.go
```

*(Only one source file is present in this snippet; additional files may exist elsewhere.)*

---

## Edge Cases for Launching the Application

| Scenario | How to Run |
|----------|------------|
| **Local development** | `go run ./cmd/qos` – starts the server on the default Unix socket. |
| **Production build** | `go build -o bin/qos ./cmd/qos && ./bin/qos` – builds a binary and runs it. |
| **Custom config path** | Pass an environment variable or flag that sets `app.ConfigPath`; e.g., `export APP_CONFIG_PATH=./config.yaml && go run ./cmd/qos`. |
| **Different endpoint** | Override the default URI via `cfg.Endpoint` in the YAML file; the listener will bind to that address. |

---

## Code‑level Relations

1. **Configuration Loading** – `configor.Load(cfg, app.ConfigPath)` populates a local `Config` struct.  
2. **Logger Creation** – `logging.BuildLogger(cfg.Logging)` produces a *zap* logger which is injected into the context via `ctxlog.WithLogger`.  
3. **Service Instantiation** –  
   - `remoteQOS := gpu.NewRemoteTuner(...)` creates a GPU‑tuning service.  
   - `remoteInit := sysinit.NewInitService(cfg.SysInit)` prepares system‑initialization logic.  
   - These services are registered on an *xgrpc* server (`xgrpc.NewServer`) and served in a dedicated goroutine.  
4. **Network Listener** – The endpoint URI is parsed, any existing Unix socket at that path is unlinked, and `net.Listen` creates the listener.  
5. **Optional PTY Server** – If `cfg.SecShell` exists, a PTY server (`secsh.NewRemotePTYServer`) watches the keystore directory for changes and runs concurrently in the same errgroup.

---

## Observations & Potential Dead Code

* The file references `golang.org/x/sync/errgroup` and `golang.org/x/sys/unix`; ensure these modules are vendored or available in the module graph.  
* No explicit TODO comments were found, suggesting the implementation is complete.  
* If any of the optional services (`remoteTuner`, `remoteInit`) fail to register, the errgroup will surface an error; consider adding more robust error handling if needed.

---