# NPP (NAT Punching Protocol) Package Summary

This package implements a NAT punching protocol (NPP) designed to establish direct connections between peers behind Network Address Translation (NAT). It supports both TCP and QUIC protocols, with fallback mechanisms using relay servers when direct connections fail. The package leverages a rendezvous server for address exchange and coordination.

**Configuration:**

*   **`config.go`:** The `Config` struct defines the overall configuration, including nested configurations for `rendezvous` and `relay` components. YAML-based configuration loading is expected.
    *   `rendezvous`: Configuration for the rendezvous server (endpoints, credentials).
    *   `relay`: Configuration for relay servers (endpoints, concurrency).
    *   `backlog`, `min_backoff_interval`, `max_backoff_interval`: Integer and duration parameters for controlling the module's behavior.
*   **`options.go`:** The `Option` function type allows configuring the listener or dialer with options such as rendezvous server, relay servers, logger, and backoff intervals.

**Environment Variables:**

*   `SONM_ENABLE_QUIC`: Enables QUIC support.

**Files and Structure:**

```
npp/
├── common.go
├── config.go
├── conn.go
├── dial.go
├── error.go
├── listener.go
├── metrics.go
├── net.go
├── nppc/
│   └── id.go
├── options.go
├── puncher.quic.go
├── puncher.tcp.go
├── relay/
│   ├── client.go
│   ├── config.go
│   ├── errors.go
│   ├── frame.go
│   ├── frame_test.go
│   ├── hashring.go
│   ├── hashring_test.go
│   ├── logging.go
│   ├── metrics.go
│   ├── monitor.go
│   ├── options.go
│   └── server.go
├── rendezvous/
│   ├── client.go
│   ├── config.go
│   ├── options.go
│   ├── peer.go
│   └── server.go
└── rv.go
```

**Key Components:**

*   **`dial.go`:** Implements the dialing logic, attempting direct TCP, QUIC (if enabled), standard NPP, and relayed connections.
*   **`listener.go`:** Manages the listener, handling TCP, QUIC, and relay connections concurrently.
*   **`puncher.quic.go` & `puncher.tcp.go`:** Implement the NAT punching logic for QUIC and TCP protocols, respectively.
*   **`rendezvous/`:** Contains code for interacting with the rendezvous server for address resolution.
*   **`relay/`:** Contains code for relay server functionality, providing fallback connections when direct punching fails.
*   **`metrics.go`:** Collects connection statistics using Prometheus and Go-Metrics.

**Edge Cases (Launch):**

*   The application can be launched with or without QUIC support, controlled by the `SONM_ENABLE_QUIC` environment variable.
*   The configuration file (YAML) must be properly formatted and contain valid rendezvous and relay server endpoints.
*   The rendezvous server must be reachable for address resolution.
*   Relay servers must be available if direct connections fail.

**Unclear Places/Dead Code:**

*   The `nppc/id.go` file is not described in the provided summaries, its purpose is unclear.
*   The TODO comments in `puncher.quic.go` and `listener.go` suggest incomplete or future work.
*   The `rv.go` file is not described in the provided summaries, its purpose is unclear.