# Package `npp`

`npp` is a small but complete library that implements a NAT‑punching protocol with support for TCP, QUIC and relay back‑ends.  
The package exposes three main building blocks:

| Component | Purpose |
|-----------|----------|
| **Config** – `config.go` | Holds the high‑level configuration (rendezvous/relay endpoints, backlog, back‑off). |
| **Options** – `options.go` | Provides a fluent API for wiring together the various punchers and relay listeners. |
| **Dialer / Listener** – `dial.go`, `listener.go` | Implements the client side (`DialContext`) and server side (`AcceptContext`). |

The remaining files provide low‑level helpers (`connResult`, `net.go`, etc.) and the concrete implementations of the four NAT punchers (TCP, QUIC, relay).

---

## File structure

```
npp/
├─ common.go
├─ config.go
├─ conn.go
├─ dial.go
├─ error.go
├─ listener.go
├─ metrics.go
├─ net.go
├─ options.go
├─ puncher.quic.go
├─ puncher.tcp.go
├─ relay/
│  ├─ client.go
│  ├─ config.go
│  ├─ errors.go
│  ├─ frame.go
│  ├─ frame_test.go
│  ├─ hashring.go
│  ├─ hashring_test.go
│  ├─ logging.go
│  ├─ metrics.go
│  ├─ monitor.go
│  ├─ options.go
│  └─ server.go
├─ rendezvous/
│  ├─ client.go
│  ├─ config.go
│  ├─ options.go
│  ├─ peer.go
│  └─ server.go
└─ rv.go
```

---

## Environment variables, flags and command‑line arguments

| Variable / flag | Description |
|------------------|-------------|
| `SONM_ENABLE_QUIC` | When set to a non‑empty string, the dialer will try the QUIC puncher first. |
| `nppBacklog` (option) | Size of the internal backlog queue for pending connections. |
| `nppMinBackoffInterval`, `nppMaxBackoffInterval` | Back‑off durations used by the dialer and listener. |
| `nppRelayConcurrency` | Number of concurrent relay listeners that will be spawned. |

The package can be started in two ways:

* **Client** – call `DialContext(ctx, network, laddr, raddr)` from a caller (e.g., an application or test harness).  
  The function will try direct TCP → QUIC → NPP → relay in that order and return the first successful connection.

* **Server** – create a `Listener` with `NewListener(opts...)`, then call `AcceptContext(ctx)` to pull connections from any of the three internal channels (direct, puncher, relay).  
  The listener will automatically spawn goroutines for listening on TCP, QUIC and relay sockets.

---

## Key code entities and their relations

| Entity | Where defined | How it is used |
|--------|----------------|-----------------|
| `Config` (`config.go`) | Holds `Rendezvous`, `Relay`, `Backlog`, `MinBackoffInterval`, `MaxBackoffInterval`. | Passed to `Dialer.NewDialer()` and `Listener.NewListener()`. |
| `Option` (type alias in `options.go`) | Functional option that mutates an internal `options` struct. | Used by `WithRendezvous`, `WithRelay`, `WithLogger`, etc. |
| `connResult` (`conn.go`) | Small wrapper around a `net.Conn` + error. | Returned from all punchers and used in the listener’s multiplexing logic. |
| `nppConn` (in `dial.go`) | Holds a `net.Conn`, source type (`connSource`) and duration. | Created by each dialer method (`dialDirect`, `dialQUICNPP`, …). |
| `puncherClientFactory`, `puncherServerFactory` (in `options.go`) | Function types for creating punchers. | Passed to the options builder; used in `WithRendezvous`. |
| `relay.Dialer` (`relay/client.go`) | Implements a relay‑based dialer that can be reused by the main dialer. | Created by `WithRelay`; used in `dialRelayed`. |
| `rendezvous.Client` (`rv.go`) | Wraps a gRPC client for rendezvous servers (TCP or QUIC). | Used by all punchers to discover remote peers. |

The flow of a connection is:

1. **Dialer**  
   * `DialContext()` → `dialContextExt()` → one of the four methods (`dialDirect`, `dialQUICNPP`, `dialNPP`, `dialRelayed`).  
   * Each method creates an `nppConn` with a specific source type and pushes it into the dialer’s metrics map.

2. **Listener**  
   * Accepts raw TCP connections via `listen()`.  
   * Periodically recreates QUIC and TCP punchers (`listenQUIC`, `listenPuncher`).  
   * Relay listeners are spawned in `runRelayListeners()` / `listenRelay`.  
   * All three streams converge into the listener’s internal channels; `AcceptContext()` pulls from them.

3. **Metrics**  
   * `dial.go` and `listener.go` expose a `dialMetrics` map keyed by address string.  
   * `metrics.go` provides helper structs (`gaugeWrapper`, `meterWrapper`, `histogramWrapper`) that convert the metrics into Prometheus‑compatible named metrics.

---

## Edge cases / launch scenarios

| Scenario | How to trigger |
|----------|----------------|
| **Direct TCP only** – set `SONM_ENABLE_QUIC=0` and configure options so that only `dialDirect()` is used. |
| **QUIC first** – export `SONM_ENABLE_QUIC=true`; the dialer will try QUIC before falling back to NPP or relay. |
| **Relay‑only** – provide a relay endpoint in `Config.Relay` and set `nppBacklog=0`. The dialer will skip direct/QUIC/NPP and use only `dialRelayed()`. |
| **Listener with multiple relays** – set `nppRelayConcurrency>1`; the listener will spawn that many goroutines for relay connections. |

---

## Summary

`npp` is a modular NAT‑punching framework that:

* Reads its configuration from a YAML file (via the tags in `Config`).  
* Builds a dialer and a listener with optional QUIC support, relay fallback and configurable back‑off.  
* Uses small helper types (`connResult`, `nppConn`) to keep connection handling tidy.  
* Exposes metrics that can be scraped by Prometheus or logged via Zap.

The package is ready for use as a library; an application can simply create a dialer, call `DialContext()`, and then start a listener with `NewListener()` to accept incoming connections.