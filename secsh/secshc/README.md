# secshc Package Summary

The `secshc` package implements a secure remote pseudo-terminal (PTY) connection using Ethereum authentication and Network Peer Protocol (NPP). It facilitates executing commands on a remote host via gRPC, streaming output back to the local terminal in real time. The core functionality revolves around establishing a TLS-encrypted channel with certificate rotation for enhanced security.

**Project Package Structure:**

```
secsh/secshc/
├── config.go       # Configuration loading (YAML)
├── protocol.go     # Protocol identifier ("secexec")
└── term.go         # Remote PTY logic, terminal manipulation, gRPC execution
```

**Configuration:**

*   **Config File Path:** Specified as an argument to `NewRPTYConfig`. YAML format expected with "ethereum" and "npp" sections.
*   **Environment Variables:** None explicitly used in the provided code snippets.
*   **Command-Line Arguments:** Not directly exposed; configuration is loaded from a file.

**Edge Cases (Launch):**

The package appears to be designed for integration into a larger application rather than standalone execution. Launching it requires providing a valid YAML config file path and ensuring that the remote host is accessible via NPP with proper Ethereum credentials configured in the `RPTYConfig`.  Errors during configuration loading or connection establishment will prevent successful operation.

**Code Relations & Unclear Areas:**

*   `config.go` provides settings for authentication (Ethereum) and transport (NPP).
*   `protocol.go` defines a simple protocol identifier, likely used in gRPC calls.
*   `term.go` orchestrates the entire process: loading keys, establishing connections, executing commands, and streaming output.

The interaction between `RemotePTYClient` (gRPC stub) and the remote host is not fully visible without additional context. The exact implementation of certificate rotation within NPP is also unclear from these snippets.  The use of a NOP Zap logger suggests that logging may be disabled or handled elsewhere in the larger application.