# Package `xnet`

`util/xnet/` implements a small networking toolkit for Go applications.  
It provides:

* **TCP listeners** that bind to all local loop‑back IPs (`listener.go`);
* **QUIC listeners** that wrap the *quic-go* stack and expose convenient connection objects (`quic.go`);
* **IP resolution helpers** that discover loop‑back addresses and fetch the host’s public IP (`resolve.go`).

The package is intended to be used by a higher‑level application (e.g. a CLI server) that needs to listen on multiple local interfaces or to accept QUIC traffic.

---

## File structure

```
util/xnet/
├─ listener.go
├─ quic.go
└─ resolve.go
```

---

## Environment variables, flags & command‑line arguments

| Source | Description |
|--------|-------------|
| `network` (string) | Protocol used for listening (`"tcp"`, `"udp"` etc.). |
| `port` (uint16) | Port number on which to listen. |
| `tlsConfig` (*tls.Config) | TLS configuration passed to a QUIC listener. |
| `config` (*quic.Config) | Configuration options for the QUIC stack. |

These values are supplied directly to the exported functions:

* `ListenLoopback(network string, port uint16)` – returns a slice of `net.Listener`s.
* `ListenPacketLoopback(network string, port uint16)` – same but for packet listeners (`net.PacketConn`).
* `ListenQUIC(network, address string, tlsConfig *tls.Config, config *quic.Config)` – creates a QUIC listener.

---

## Summary of major code parts

### 1. `listener.go`

* **BackPressureListener**  
  Wraps a standard `net.Listener` with a Zap logger and implements an exponential‑backoff accept loop (`Accept()`).

* **ListenLoopback / ListenPacketLoopback**  
  Create one or more listeners bound to all loop‑back IP addresses returned by `LookupLoopbackIP()`.  
  They validate the network string, discover local IPs via `resolve.go`, and close any partially created listeners on failure.

### 2. `quic.go`

* **`DefaultQUICConfig()`** – returns a ready‑to‑use QUIC configuration (supports GQUIC39/43/Milestone0_10_0).

* **`QUICConn` & constructor**  
  Represents a QUIC connection that bundles a stream and its session.  
  Methods `LocalAddr()`, `RemoteAddr()` and `Close()` expose the underlying addresses and close both stream and session.

* **`ListenQUIC()`** – creates a packet listener, wraps it into a `quic.Listener`, and returns a ready‑to‑accept `QUICListener`.

* **`QUICListener.Accept()`** – loops until a QUIC session is accepted, opens a new stream, and returns a fully initialized `QUICConn`.

### 3. `resolve.go`

* **`LookupLoopbackIP()`** – enumerates all loop‑back interfaces on the host, returning IPv6 addresses first followed by IPv4.

* **`ExternalPublicIPResolver`**  
  Lightweight helper that fetches the caller’s public IP via an HTTP endpoint (`http://checkip.amazonaws.com/`) and caches it for a configurable duration.  
  Methods: `NewExternalPublicIPResolver()`, `PublicIP()`, `needRefresh()`, `refresh()` and `resolve()`.

---

## Relations between code entities

* `listener.go` uses the IP list produced by `LookupLoopbackIP()` from `resolve.go`.
* `quic.go` depends on the QUIC stack (`github.com/lucas-clemente/quic-go`) and aggregates errors with `multierror`.  
  The returned `QUICListener` is a thin wrapper around `quic.Listener`; its `Accept()` method relies on `isPeerGoneErr()` to decide whether to retry.
* All three files share the same package name (`xnet`), so they can be imported together by an application.

---

## Edge cases for launching

1. **TCP server** – call `xnet.ListenLoopback("tcp", 8080)` in a main package, then iterate over the returned listeners to accept connections.
2. **UDP/QUIC server** – use `ListenPacketLoopback` or `ListenQUIC` with appropriate TLS configuration; the QUIC listener will automatically open streams on each accepted session.

---

>