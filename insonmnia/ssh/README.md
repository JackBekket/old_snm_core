# ssh Package Summary

The **ssh** component implements a lightweight SSH proxy server that forwards traffic to remote containers using the *gliderlabs/ssh* library and a custom NPP dialer.  
It is split into three files:

```
insonmnia/ssh/
├── config.go   – configuration struct for the proxy server
├── crypto.go   – helpers for creating, serializing, parsing and verifying an SSH identity
└── proxy.go   – core server logic (connection handling, NPP dialing, agent integration)
```

## Short overview

* `config.go` defines a single configuration type (`ProxyServerConfig`) that holds the listening address and an embedded NPP configuration.  
* `crypto.go` introduces the `SSHIdentity` struct and three helper functions: constructor, parser, and verifier. These are used by the proxy to sign/verify identities for remote containers.  
* `proxy.go` contains the main server implementation (`SSHProxyServer`) together with a connection handler that translates an incoming SSH session into a remote container request, forwards data streams, and logs activity.

The package can be started from any Go program that calls `NewSSHProxyServer(cfg, key, creds, market, log)` followed by `Serve(ctx)`. It expects the environment variable **`SSH_AUTH_SOCK`** to point to an active ssh‑agent socket.  

---

## Environment variables

| Variable | Purpose |
|----------|---------|
| `SSH_AUTH_SOCK` | Path of the running ssh‑agent socket (used in `proxy.go`). |

---

## Files and paths

```
insonmnia/ssh/config.go
insonmnia/ssh/crypto.go
insonmnia/ssh/proxy.go
```

---

## Relations between code entities

| File | Key types / functions | How they interact |
|------|-----------------------|-------------------|
| `config.go` | `ProxyServerConfig` | Passed to `NewSSHProxyServer`; contains NPP config used when creating the dialer. |
| `crypto.go` | `SSHIdentity`, `NewSSHIdentity`, `ParseSSHIdentity`, `Verify` | Identity is created from a private key, parsed from user strings, and verified before forwarding data in `proxy.go`. |
| `proxy.go` | `SSHProxyServer`, `connHandler`, `convertHostSigners`, `handle`, `extractMeta`, `resolve` | The server creates an NPP dialer, listens on the configured address, accepts SSH sessions, extracts meta (remote address + identity), resolves deals via market API, and forwards streams to the remote container. |

---

## Edge cases for launching

* **Configuration** – A YAML file can be unmarshaled into `ProxyServerConfig` because all fields have `yaml:` tags.  
* **SSH agent socket** – If `os.Getenv(sshAgentSockName)` is empty or the socket cannot be opened, the server will return an error; ensure that the environment variable `SSH_AUTH_SOCK` points to a valid ssh‑agent.  
* **Connection handling** – The handler spawns three goroutines for stdin/stdout/stderr forwarding; if any of them fails, the session is closed and the error is logged.  

---

## Summary of major code parts

1. **Configuration (`config.go`)** – simple struct with YAML tags.  
2. **Identity helpers (`crypto.go`)** – create, serialize, parse, and verify an Ethereum‑style SSH identity.  
3. **Server logic (`proxy.go`)** – builds a dialer, listens on TCP, accepts sessions, resolves deals, forwards streams, and logs activity.

All parts together provide a fully functional SSH proxy that can be integrated into the larger system or run as a standalone service.