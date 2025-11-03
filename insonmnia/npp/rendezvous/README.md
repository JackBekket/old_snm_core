# Rendezvous Package Summary

This package implements a bidirectional locator protocol for mutual address resolution between peers, particularly useful in NAT environments. The core functionality revolves around facilitating P2P connections by exchanging public and private network addresses. It provides server-side components for managing meetings (sessions) between clients and servers.

**File Structure:**

```
insonmnia/npp/rendezvous/
├── client.go
├── config.go
├── options.go
├── peer.go
└── server.go
```

**Configuration & Environment Variables:**

*   `config.go`: Loads configuration from YAML files, including:
    *   Server address (`Addr`)
    *   Ethereum private key path (`PrivateKey`) for authentication.  Must be a valid Ethereum key file.
    *   Logging settings (level, format).
    *   Debugging options.
*   `options.go`: Configures TLS credentials via `WithCredentials`. Requires a valid `tls.Config` instance if QUIC is enabled.

**Launch Edge Cases:**

*   Server:  Requires a valid configuration file path to load settings. If no TLS config is provided, connections will be unencrypted (discouraged).
*   Client: No specific launch edge cases beyond standard gRPC client setup.

**Key Components & Logic Flow:**

1.  **Peer Identification (`peer.go`):** Assigns unique UUID-based IDs (`PeerID`) to connected peers for tracking, especially when Ethereum addresses are insufficient.
2.  **Meeting Management (`server.go`):** The `meeting` struct manages sessions between clients and servers using mutexes for thread safety. It facilitates peer discovery by randomly selecting available servers or waiting clients.
3.  **Address Resolution (`server.go`):** Clients call the `Resolve` RPC to find a matching server, while servers publish their addresses via the `Publish` RPC. If direct connection fails (NAT), TCP punching is attempted but not fully implemented.
4.  **gRPC Integration (`server.go`):** The server uses gRPC for communication with custom interceptors for logging and authentication. Keep-alive parameters are configured to maintain active connections.

**Potential Issues & Dead Code:**

*   The `TODO` in `server.go` indicates incomplete IPv4/IPv6 compatibility handling during address resolution.
*   TCP punching is mentioned but not fully implemented, suggesting potential dead code or future work.