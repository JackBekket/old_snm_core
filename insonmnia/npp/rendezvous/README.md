# rendezvous – a gRPC‑based rendezvous service

The `rendezvous` package implements a lightweight peer‑to‑peer rendezvous
service that can be used by other components of the *insonmnia* NPP stack.
It provides:

* A thin client wrapper (`client.go`) for external callers to talk to the
  server over gRPC.
* Configuration structures and YAML loader (`config.go`).
* Functional options (`options.go`) that let a caller inject logging,
  TLS credentials, and QUIC support.
* A `Peer` abstraction (`peer.go`) that keeps an ID and a list of private
  addresses.
* The full server implementation (`server.go`) – it tracks meetings,
  exchanges peers over gRPC, serves TCP/QUIC, and exposes debugging
  endpoints.

---

## Project structure

```
insonmnia/npp/rendezvous/
├── client.go
├── config.go
├── options.go
├── peer.go
└── server.go
```

All files belong to the same package `rendezvous`.  
The public API is exposed through the following types:

| File | Public type / function |
|------|------------------------|
| `client.go` | `Client`, `NewRendezvousClient` |
| `config.go` | `ServerConfig`, `NewServerConfig` |
| `options.go` | `Option`, `WithLogger`, `WithCredentials`, `WithQUIC` |
| `peer.go` | `Peer`, `NewPeer`, `PeerID` |
| `server.go` | `Server`, `NewServer`, `Run`, `Info` |

---

## Environment / configuration

* **YAML file** – passed to `NewServerConfig(path string)`.  
  The file must contain the keys `endpoint`, `ethereum`, `logging`,
  and optionally `debug`. Example:

```yaml
endpoint: "127.0.0.1:9000"
ethereum:
  private_key_path: "/tmp/key.pem"
logging:
  level: "info"
debug:
  pprof_port: 6060
```

* **Environment variable** – the package does not read any env var directly,
  but callers may use `RZV_CONFIG_PATH` (conventionally) to pass the path
  to `NewServerConfig`.

---

## Flags / command‑line arguments

| Flag | Description |
|------|-------------|
| `-config <path>` | Path to YAML config file – used by `NewServerConfig`. |
| `-addr <tcp_addr>` | TCP address for the server (overridden by config). |
| `-enable-quic` | Enables QUIC support via `WithQUIC()`. |

The flags are not implemented in this snippet, but they can be wired into a
CLI wrapper that calls `NewServerConfig`, builds an `options` instance with
`WithLogger(...)`, `WithCredentials(...)`, and finally `NewServer`.

---

## How the application can be launched

1. **As a library** – other packages import `rendezvous` and call  
   `client := rendezvous.NewRendezvousClient(ctx, addr, creds, opts...)`.  
2. **As a standalone server** – a small `main.go` could do:

```go
func main() {
    cfg, err := rendezvous.NewServerConfig(os.Args[1])
    if err != nil { log.Fatal(err) }

    srv := rendezvous.NewServer(cfg,
        rendezvous.WithLogger(zap.NewNop()),
        rendezvous.WithCredentials(&tls.Config{...}),
        rendezvous.WithQUIC(),
    )
    srv.Run(context.Background())
}
```

The server will start three goroutines: TCP, QUIC (if enabled), and a
debug endpoint. It blocks until the context is cancelled.

---

## Relationships between code entities

* **`Client`** wraps a generated gRPC client (`sonm.RendezvousClient`) and
  keeps its connection handle.  
  `NewRendezvousClient` creates this wrapper, so callers only need to
  provide an address and optional credentials.
* **`ServerConfig`** is the root configuration object used by `Server`.  
  It contains a network address (`Addr`), an ECDSA key (`PrivateKey`) for
  signing, and logging/debug structs.  
  The internal `serverConfig` struct mirrors the YAML layout; `NewServerConfig`
  loads it into a `*ServerConfig`.
* **`options`** is a small helper that lets callers inject a logger,
  TLS credentials, and enable QUIC.  
  These options are applied in `NewServer`, which builds the gRPC server
  instance (`grpc.Server`) and registers the service.
* **`Peer`** holds an ID (`PeerID`) and a slice of private addresses.
  It is created by `NewPeer` from a gRPC peer object and a list of
  `*sonm.Addr`.  
  The `PeerID` type is just a string alias; it is used as the key in all
  meeting maps.
* **Meetings** – the server keeps a map `rv map[nppc.ResourceID]*meeting`.
  Each `meeting` contains two maps (`clients`, `servers`) that hold
  `peerCandidate`s.  
  A `peerCandidate` bundles a `Peer` and a channel used to hand over the
  matched peer during rendezvous.
* **RPC handlers** – `Resolve`, `Publish`, `ResolveAll`, and `Info`
  operate on these meetings.  
  They use helper functions (`addServerWatch`, `addClientWatch`,
  `popRandomPeerCandidate`) to move peers between maps and exchange them
  via the channel in a `peerCandidate`.

---

## Edge cases & launch options

* **QUIC support** – if `WithQUIC()` is omitted, only TCP will be served.
  The server still starts the QUIC goroutine but it will be idle until a
  client connects over UDP.  
* **Multiple meetings** – the server can handle several NPP identifiers
  simultaneously; each identifier gets its own `meeting`.  
  If a client and a server arrive at different times, they are matched
  when both sides have been added to the same meeting.
* **Server shutdown** – calling `srv.Stop()` will stop all gRPC listeners;
  it is currently not exposed in this snippet but can be wired into a
  graceful‑shutdown routine.

---

## Summary

`rendezvous` provides:

1. A client wrapper for external callers (`client.go`).  
2. YAML‑driven configuration (`config.go`) that produces a `ServerConfig`.  
3. Functional options (`options.go`) to customize logging, TLS credentials,
   and QUIC support.  
4. A lightweight peer abstraction (`peer.go`).  
5. The full server implementation (`server.go`) – it tracks meetings,
   exchanges peers over gRPC, serves TCP/QUIC, and exposes debugging
   endpoints.

The package can be used as a library or run as a standalone service; the
public API is intentionally small so that callers only need to provide a
configuration file, optional options, and then call `Run`.