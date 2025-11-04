# secshc – Remote PTY client package

**Short summary**  
The `secshc` package provides a lightweight remote pseudo‑terminal (PTY) client that can connect to an Ethereum/NPP node, execute commands over gRPC and stream terminal I/O back to the local console.  The three source files define:  

* **config.go** – YAML configuration loader (`RPTYConfig`) for Ethereum & NPP settings.  
* **protocol.go** – a single constant that identifies the protocol name used when dialing an NPP node.  
* **term.go** – the core logic: construction of a `RemotePTY`, terminal handling helpers, and command execution over gRPC.

---

## Project package structure

```
secsh/
└─ secshc/
   ├─ config.go
   ├─ protocol.go
   └─ term.go
```

---

## Environment variables / configuration sources  

| Variable | Purpose | Default / Example |
|----------|---------|-------------------|
| `RPTY_CONFIG_PATH` | Path to the YAML file that contains `"ethereum"` and `"npp"` sections. | `./config.yaml` (used by `NewRPTYConfig`) |

---

## Flags & command‑line arguments  

The package itself does not expose a CLI flag set, but the public API can be invoked from a higher‑level main program:

| Flag / Arg | Description |
|------------|-------------|
| `-c <path>` | Path to the YAML config file (passed to `NewRPTYConfig`). |
| `-a <addr>` | Remote host address for the PTY connection (`auth.Addr` used in `RemotePTY.Run`). |

---

## How the application can be launched  

1. **As a library** – import `"github.com/sonm-io/core/secshc"` and call:

   ```go
   cfg, _ := secshc.NewRPTYConfig("./config.yaml")
   pty := secshc.NewRemotePTY(cfg)
   pty.Run(ctx, auth.Addr{Host:"127.0.0.1", Port:2222})
   ```

2. **As a CLI** – create a small `main.go` that parses the flags above and calls the same sequence.

---

## Detailed code walk‑through  

### 1️⃣ `config.go` – Configuration loader  
* Defines `RPTYConfig` with two fields (`Eth`, `NPP`) tagged for YAML.  
* `NewRPTYConfig(path string)` loads a file into that struct using `github.com/jinzhu/configor`.  
* The function returns a pointer to the config or an error.

### 2️⃣ `protocol.go` – Protocol constant  
* Declares `const Protocol = "secexec"`.  
* This value is used in `term.go` when creating an NPP dialer (`npp.Option{Protocol: secshc.Protocol}`).

### 3️⃣ `term.go` – Core PTY logic  

| Entity | Purpose |
|--------|---------|
| `RemotePTY` struct | Holds a config pointer and an ECDSA private key. |
| `NewRemotePTY(cfg *RPTYConfig)` | Loads the private key from the config, creates a new instance. |
| `Run(ctx context.Context, addr auth.Addr) error` | Main entry point: dials the remote host, builds a gRPC client, obtains a banner, and enters an interactive loop that reads terminal input, executes it on the remote PTY, and prints output. |
| `withRaw(fn func(ttyState *terminal.State) error)` | Helper to switch the local terminal into raw mode for each command execution. |
| `withRestored(ttyState *terminal.State, fn func())` | Restores the terminal state after a command and re‑enters raw mode. |
| `executeCmd(ctx context.Context, remotePTY sonm.RemotePTYClient, line string, stdin *stdinPipe)` | Sends a single command to the remote PTY via gRPC, concurrently reads output from the stream and writes input from a local pipe. |
| `stdinPipe` struct & helpers (`newStdinReader`, `Read`, `Write`, `ReadContext`) | Simple channel‑based I/O abstraction for stdin data. |
| `byteOrError` struct | Small wrapper used by `stdinPipe`. |

#### Flow of execution  

1. **Dial** – `Run` creates an NPP dialer with the protocol name from `protocol.go`.  
2. **Client** – Builds a gRPC client (`xgrpc.Client`) and obtains a `sonm.RemotePTYClient`.  
3. **Banner** – Calls `remotePTY.Banner()` to fetch a welcome string.  
4. **Terminal loop** – For each line read from the local terminal, it calls `executeCmd`, which writes the command to the remote PTY and streams back the output.  

---

## Relations between code entities  

* `config.go` → `term.go`: The config struct is passed into `NewRemotePTY`; the loaded private key is used when dialing NPP.  
* `protocol.go` → `term.go`: The constant `Protocol` is referenced in the dialer options (`npp.Option{Protocol: secshc.Protocol}`).  
* `term.go` uses `xgrpc.Client`, `auth.Addr`, and `npp.Config` from other packages, tying together Ethereum & NPP configuration with gRPC communication.  

---

## Edge cases / potential launch scenarios  

| Scenario | What to check |
|----------|---------------|
| **Missing config file** – `NewRPTYConfig` will return an error; ensure the path is correct or provide a default fallback. |
| **Invalid Ethereum key** – The private key must be present in the YAML under `"ethereum"`; otherwise `NewRemotePTY` fails. |
| **Connection timeout** – If dialing fails, `Run` logs the error and returns it to the caller. |

---

## Summary of what the package does  

The `secshc` package loads a YAML configuration for Ethereum & NPP, creates a remote PTY client over gRPC, and provides an interactive terminal loop that executes commands on a remote host while streaming output back to the local console.