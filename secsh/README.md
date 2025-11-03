```markdown
# secsh Package Summary: Secure Shell with Ethereum Integration

The `secsh` package provides a secure shell environment integrated with Ethereum authentication and policy enforcement. It allows remote execution of commands within a controlled sandbox, leveraging Seccomp profiles for enhanced security. The core functionality revolves around authenticating users via Ethereum addresses (whitelisted in configuration) and enforcing network policies defined by an external Network Policy Provider (NPP).

**Configuration:**

The package relies heavily on YAML-based configuration loaded at startup:

*   `SecExecPath`: Path to the `secexec` binary, used for sandboxed command execution.
*   `SeccompPolicyDir`: Directory containing Seccomp policy files that restrict system calls within executed commands.
*   `AllowedKeys`: List of Ethereum addresses authorized to access remote PTY services.  Compromised keys are explicitly excluded.
*   `Eth.KeyPath`: Path to the private key used for TLS certificate generation and potentially other cryptographic operations (e.g., signing requests).
*   `NPP.*`: Network Policy Provider settings, including backlog size and backoff intervals.

**Workflow:**

1.  **Authentication:** Clients connect via gRPC over TLS. Authentication is performed by verifying the client's Ethereum address against the `AllowedKeys` list.
2.  **Command Execution:** Authenticated clients can request command execution through the `Exec` service. The package parses piped commands, resolves executables using `exec.LookPath`, and executes them within a sandboxed environment enforced by Seccomp policies.
3.  **Policy Enforcement:** Network access is controlled via NPP, ensuring that only authorized traffic reaches the remote PTY session.
4.  **Dynamic Whitelisting (TODO):** The package includes an ACL update loop (`runACLUpdateLoop`) intended to fetch updated whitelists from a remote source but currently lacks implementation details.

**File Structure:**

*   `banner.go`: Generates a system banner with host information, load averages, disk usage, and logged-in users.
*   `config.go`: Defines the `Config` struct for loading external configuration (YAML).
*   `exec.go`: Handles command parsing and execution within a sandboxed environment using `os/exec`.
*   `secshc/*`: Contains protocol definitions and terminal handling logic.
*   `server.go`: Implements the gRPC server, TLS certificate rotation, and network listener setup via NPP.
*   `service.go`: Defines the remote PTY service with methods for banner generation (`Banner`) and command execution (`Exec`).
*   `watch.go`: Monitors a directory for existence (used for dynamic configuration updates).

**Dependencies:**

The package relies on several external libraries:

*   `github.com/sonm-io/core/*`: Core components from the SONM ecosystem, including Ethereum account management and NPP integration.
*   `gopsutil/*`: System information retrieval utilities (CPU load, memory usage, disk space).
*   `go.uber.org/zap`: Structured logging framework.

**Potential Issues:**

The reliance on external configuration files makes the package vulnerable to misconfiguration or malicious input. The lack of detailed error handling in some functions could lead to unexpected behavior.  The unimplemented ACL update loop (`runACLUpdateLoop`) represents a potential security risk if not properly addressed.
<end_of_output>
```