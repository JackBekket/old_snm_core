## SSH Proxy Package Summary

This package implements an SSH proxy server that leverages blockchain-based identity resolution and network tunneling via NPP (Network Proxy Protocol). The proxy authenticates users via SSH agent keys, extracts metadata from usernames (Deal ID and Task ID), resolves remote endpoints using a blockchain market API, and forwards traffic through an NPP tunnel.

**Configuration:**

*   **Environment Variables:** `SSH_AUTH_SOCK` (path to SSH agent socket).
*   **Configuration File:** YAML format with `endpoint` (SSH server address) and `npp` (nested `npp.Config` structure).
*   **Command-Line Arguments:** None explicitly defined in the provided code.

**Edge Cases:**

*   The server expects a running SSH agent with loaded keys. If no agent is available or the socket is invalid, authentication will fail.
*   The blockchain market API must be accessible for identity resolution. If the API is unreachable, connections will be rejected.
*   The NPP dialer must be configured correctly to establish tunnels to remote endpoints.

**Project Package Structure:**

```
insonmnia/ssh/
├── config.go
├── crypto.go
└── proxy.go
```

**Relationships:**

*   `config.go` defines the `ProxyServerConfig` struct, which holds the server's configuration parameters.
*   `crypto.go` provides functions for creating and verifying SSH identities based on Ethereum addresses and ECDSA signatures.
*   `proxy.go` implements the core SSH proxy server logic, integrating with SSH agent, blockchain market API, and NPP.

**Unclear Places/Dead Code:**

*   The TODOs in `proxy.go` ("Activate relay, but for now disable for rendezvous testing" and "stdout/stderr intermixing is possible. How to get with it?") suggest incomplete or experimental features.
*   The reliance on SSH agent keys for authentication introduces a dependency on external key management.