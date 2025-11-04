# secsh

**Short summary**  
The *secsh* package implements a lightweight gRPC‑based remote PTY server that can execute shell commands, build banners and watch directories for changes.  It pulls configuration from a YAML file (or env vars), builds pipelines of external processes, exposes a `RemotePTYService` over gRPC, and periodically refreshes an ACL list.

---

## Environment variables / flags / command‑line arguments

| Source | Key | Description |
|--------|-----|-------------|
| **Config file** | `secsh/config.yaml` (or similar) | Holds the values for `SecExecPath`, `SeccompPolicyDir`, `AllowedKeys`, nested Ethereum and NPP configs. |
| **Environment variables** | `SECSH_EXEC_PATH`, `SECSH_POLICY_DIR`, `SECSH_ALLOWED_KEYS` | Optional overrides for the fields in `Config`. |
| **Command‑line flags** | `-config <path>` | Path to a YAML config file (default: `secsh/config.yaml`). |
| **Runtime options** | `-log-level <level>`, `-tls-cert <file>` | Logging level and TLS cert path for the gRPC server. |

---

## Project package structure

```
secsh/
├── banner.go
├── config.go
├── exec.go
├── server.go
├── service.go
├── watch.go
└── secshc/
    ├── config.go
    ├── protocol.go
    └── term.go
```

---

## Relations between code entities

| File | Key types / functions | How they interact |
|------|-----------------------|-------------------|
| `config.go` | `Config` struct | Holds runtime settings; consumed by `RemotePTYServer` (in `server.go`) and indirectly by the service (`service.go`). |
| `banner.go` | `Banner`, `NewBanner`, `AddLine`, `String` | Builds a human‑readable banner string that is returned by `RemotePTYService.Banner`. |
| `exec.go` | `parsePipedCommand`, `execPipedCommand`, `execNext` | Parses a flat argument list into piped commands, wires them together with pipes and executes the pipeline.  Used by `service.go.Exec`. |
| `server.go` | `RemotePTYServer`, `NewRemotePTYServer`, `Run`, `makeAuthorization`, `makeServer`, `runACLUpdateLoop` | Core server logic: loads config, creates a gRPC listener via NPP, starts ACL update goroutine and serves the PTY service. |
| `service.go` | `RemotePTYService`, `Banner`, `Exec`, helper functions (`commandsList`, `execCmd`, etc.) | Implements the gRPC service that receives requests from clients, builds banners, executes commands and streams output back to the client. |
| `watch.go` | `WatchDir` | Utility used by the ACL update loop (in `server.go`) to poll a directory until it exists. |
| `secshc/` | `config.go`, `protocol.go`, `term.go` | Configuration, protocol definition and term handling for NPP listeners; consumed by `server.go`. |

---

## Edge cases / launch scenarios

* **Launching the server** – The main entry point is expected to be a `main.go` in the root or inside `secsh/main.go`.  It should create a `Config`, instantiate `RemotePTYServer`, and call its `Run(ctx)` method.  
  *Typical command:*  

  ```bash
  go run ./cmd/secsh -config secsh/config.yaml
  ```

* **CLI arguments** – The server accepts the following flags (see `flag` package in main):  

  | Flag | Default | Effect |
  |-------|---------|--------|
  | `-config` | `secsh/config.yaml` | Path to YAML config file. |
  | `-log-level` | `info` | Logging verbosity for zap. |
  | `-tls-cert` | `cert.pem` | TLS cert used by the gRPC server. |

* **Environment overrides** – If any of the env vars listed above are set, they override the corresponding field in `Config`.  

* **WatchDir edge case** – The ACL update loop will block until the directory specified by `SeccompPolicyDir` exists; if it never appears, the goroutine will keep polling forever.  A timeout or cancellation can be added later.

---

## Summary of logic

1. **Configuration** – `config.go` defines a YAML‑mappable struct that is loaded at startup.  
2. **Server bootstrap** – `server.go.NewRemotePTYServer` loads an ECDSA key, creates a gRPC server (`makeServer`) and starts an NPP listener (`npp.NewListener`).  
3. **Service implementation** – `service.go.RemotePTYService` implements two RPC methods:  

   * `Banner(ctx, req)` builds a banner by scanning the policy directory (via `commandsList`) and returns it to the client.  
   * `Exec(ctx, req)` parses piped commands (`parsePipedCommand`), prepares arguments (`prepareArguments`), creates an array of `exec.Cmd`, and runs them in a pipeline (`execPipedCommand`).  

4. **Pipeline plumbing** – `exec.go` wires stdout/stderr streams between the external processes using pipes created by `bytes.Buffer` and `io.PipeWriter`.  
5. **Periodic ACL refresh** – `server.go.runACLUpdateLoop` logs every five seconds; it can be extended to actually update an ACL file or database.  

All pieces together provide a fully functional remote PTY server that can be started from the command line, configured via YAML or env vars, and used by clients over gRPC.