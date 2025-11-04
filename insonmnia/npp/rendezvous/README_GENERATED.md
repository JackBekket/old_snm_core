## Rendezvous Package Summary

This package implements a rendezvous protocol for peer-to-peer address resolution, facilitating connections between nodes behind NATs or with uncertain connectivity. It leverages gRPC for communication and supports TLS/QUIC for secure connections. The core logic revolves around matching clients and servers based on a shared identifier, exchanging private and public addresses, and attempting direct connections or relaying if necessary.

**Configuration:**

*   **YAML Configuration File:** The primary configuration source, loaded via `github.com/jinzhu/configor`. Defines server settings (address, TLS), Ethereum account details, logging, and debugging options.
*   **Ethereum Private Key:** Required for TLS/QUIC authentication. Loaded from the YAML config.
*   **TLS Configuration:** Optional but recommended for secure connections. Defined in the YAML config.
*   **QUIC Support:** Enabled via functional options (`WithQUIC`), requiring TLS credentials.
*   **Logging:** Configurable via `zap` logger instance.

**Files:**

*   `client.go`: Wraps the generated gRPC client, adding a `Close()` method for explicit connection termination.
*   `config.go`: Handles loading and validating server configuration from YAML.
*   `options.go`: Implements functional options for configuring the server (logger, credentials, QUIC).
*   `peer.go`: Defines the `Peer` struct, including a unique `PeerID` generated using UUIDs.
*   `server.go`: Implements the core rendezvous server logic, handling gRPC connections, address resolution, and meeting management.

**Edge Cases:**

*   The server can be launched with or without TLS/QUIC. Disabling TLS is discouraged for production environments.
*   The server requires a valid Ethereum private key if QUIC is enabled.
*   The server can be configured to listen on specific TCP addresses.

**Unclear Places/Dead Code:**

*   The `TODO` in `server.go` regarding IPv6 resolution suggests incomplete handling of dual-stack environments.
*   The reliance on `xnet.ExternalPublicIPResolver` may introduce external dependencies and potential failure points.