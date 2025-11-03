# insonmnia/auth

This package implements gRPC authorization mechanisms based on Ethereum addresses and dynamic access control using time-to-live credentials. It provides a flexible router for registering custom authorizations, including nil (allow all), deny (deny all), and transport credential checks against configured or dynamically updated Ethereum wallets. The core logic revolves around extracting wallet information from the gRPC context and comparing it against authorized peers to enforce access policies.

**Configuration:**

*   **Environment Variables:** None explicitly defined in the provided code, but external configuration could be loaded via environment variables for settings like log levels or default authorization rules.
*   **Flags/Cmdline Arguments:** Not applicable as this is a library package without direct command-line execution.
*   **Files/Paths:** The `auth` directory contains all relevant source files (`addr.go`, `auth.go`, `common.go`) and tests (`addr_test.go`, `common_test.go`). YAML configuration could be used for dynamic authorization rules, but the parsing logic is not shown in this snippet.

**Launch Edge Cases:**

This package does not have a main entry point; it's designed to be integrated into gRPC servers or clients as part of an authentication pipeline. The behavior depends entirely on how the `AuthRouter` and related components are configured within the calling application. Incorrect configuration (e.g., allowing all traffic with nil authorization) could lead to security vulnerabilities.

**Project Package Structure:**

```
insonmnia/auth/
├── addr.go          # Unified Ethereum/Network address handling
├── addr_test.go     # Tests for `addr.go`
├── auth.go          # Core gRPC authorization router and logic
├── common.go        # Authentication utilities (wallet extraction, comparison)
└── common_test.go   # Tests for `common.go`
```

**Code Relations & Unclear Places:**

The package relies heavily on Ethereum address handling from the `github.com/ethereum/go-ethereum/common` library. The dynamic authorization component (`AnyOfTransportCredentialsAuthorization`) introduces complexity with its TTL-based access control, which could lead to race conditions if not properly synchronized. The hardcoded insecure key in `common.go` is a potential security risk and should be removed or replaced with proper configuration management.

The exact implementation of wallet extraction from the gRPC context (`FromContext`, `ExtractWalletFromContext`) is unclear without seeing the full code, but it likely involves inspecting TLS credentials or custom metadata. The relationship between `EthAuthInfo` and the underlying transport layer (TLS) is also not fully defined in this snippet.