# relay

## Overview  
The *relay* package implements a lightweight TCP‑based node discovery and handshaking layer that is used by the NPP (Network Peer‑Peer) server.  It provides:

* **Client/Server logic** – dialing, listening, and exchanging protobuf frames (`client.go`, `server.go`).  
* **Configuration handling** – YAML loading into a `ServerConfig` struct (`config.go`).  
* **Error handling** – small helper errors for the protocol (`errors.go`).  
* **Frame serialization** – generic send/recv helpers that are unit‑tested (`frame.go`, `frame_test.go`).  
* **Continuum & node tracking** – a hash ring that maps Ethereum addresses to human‑readable nodes (`hashring.go`, `hashring_test.go`).  
* **Logging adapter** – turns Zap logs into byte streams for the network layer (`logging.go`).  
* **Metrics collection** – per‑node counters and overall uptime (`metrics.go`).  
* **Monitoring & gRPC server** – exposes cluster membership, metrics, and meeting information over gRPC (`monitor.go`).  

The package is intended to be started as a command line binary that creates a `Server` instance from a YAML file, listens on the configured TCP endpoint, and serves both raw TCP connections (for handshakes) and gRPC monitoring.

---

## Environment variables / flags

| Variable / Flag | Purpose |
|-----------------|---------|
| `tcpKeepAliveInterval` (`client.go`) | Keep‑alive interval for dialing a relay server. |
| `bufferSize` (`options.go`) | Size of the internal TCP buffer (default 32 KiB). |
| `log` (`options.go`, `config.go`) | Zap logger instance used throughout the package. |

---

## Command line arguments

* **`-c <path>`** – path to a YAML configuration file that is parsed by `NewServerConfig`.  
* **`-p <port>`** – TCP port on which the relay listens (used in `formatEndpoint`).  
* **`--log-level=<level>`** – optional log level for the Zap logger.  

The binary can be invoked as:

```bash
$ npp-relay -c config.yaml -p 9000
```

---

## File structure

```
insonmnia/npp/relay/
├─ client.go
├─ config.go
├─ errors.go
├─ frame.go
├─ frame_test.go
├─ hashring.go
├─ hashring_test.go
├─ logging.go
├─ metrics.go
├─ monitor.go
├─ options.go
└─ server.go
```

---

## Relations between code entities

| File | Key types / functions | Relationship |
|------|-----------------------|--------------|
| `client.go` | `Dial`, `Listen`, `newClient`, `discover`, `dial`, `accept`, `handshake` | Provides the low‑level client side of a handshaking round. |
| `server.go` | `NewServer`, `Serve`, `processConnection`, `processDiscover`, `processHandshake` | Top‑level server that uses the client helpers to accept connections and hand them off to meetings. |
| `hashring.go` | `continuum`, `Node`, `newContinuum`, `Add`, `Remove`, `Track`, `GetNode` | Maintains a hash ring of nodes; used by `processDiscover` to look up the target node for a handshake. |
| `monitor.go` | `Cluster`, `Metrics`, `Info`, `Serve` | Exposes cluster membership and metrics over gRPC; called from `server.go`. |
| `metrics.go` | `metrics`, `newMetrics`, `NetMetrics`, `Dump` | Stores per‑node counters that are read by the monitor. |
| `frame.go` / `frame_test.go` | `sendFrame`, `recvFrame` | Generic frame serialization used by both client and server. |
| `config.go` | `NewServerConfig` | Loads YAML into a `ServerConfig`; feeds data to `NewServer`. |
| `options.go` | `Option`, `WithLogger` | Functional options for configuring the relay (buffer size, logger). |

The flow is:  
1. `NewServerConfig` → `NewServer` → `Serve` starts TCP and gRPC servers.  
2. Incoming connection → `processConnection` → either `processDiscover` or `processHandshake`.  
3. `processDiscover` uses the continuum to find a node, sends back a discovery frame.  
4. `processHandshake` creates a unique connection ID, registers it with the meeting hall, and hands over to a `meetingHandler`.  

---

## Edge cases for launching

| Scenario | How to launch |
|----------|----------------|
| **Single node** – run `npp-relay -c config.yaml -p 9000`; the server will listen on `localhost:9000`, accept connections, and expose metrics via gRPC. |
| **Multiple nodes** – start several instances with distinct ports; each will load its own YAML file (or share one with different `Endpoint` values). The continuum will map addresses to node names, allowing handshakes between them. |
| **Restart / graceful shutdown** – the server’s `Close` method stops the gRPC server and TLS rotator; it can be called from a signal handler or via a systemd unit. |

---

## Summary of logic

* The package reads a YAML config file into a `ServerConfig`.  
* A `Server` instance is created with options (buffer size, logger).  
* It listens on the configured TCP endpoint and serves gRPC monitoring concurrently.  
* Incoming connections are processed by either discovery or normal handshaking; frames are serialized via `sendFrame/recvFrame`.  
* The continuum keeps track of nodes; metrics are collected in a thread‑safe map and exposed over gRPC.  

All pieces fit together to provide a minimal but functional relay server that can be used as part of the NPP stack.

---

**<end_of_output>**