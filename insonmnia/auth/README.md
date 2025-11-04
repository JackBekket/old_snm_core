# `auth` Package Summary

This package implements gRPC authorization logic, focusing on Ethereum address-based authentication. It provides a flexible authorization router, dynamic rule management, and utility functions for extracting wallet addresses from gRPC contexts.

**Configuration:**

*   **Environment Variables:** None explicitly defined in the provided code.
*   **Flags/Cmdline Arguments:** None.
*   **Files/Paths:** The package is structured as follows:
    *   `addr.go`: Defines the `Addr` struct for unified address handling (Ethereum and network).
    *   `addr_test.go`: Tests the `Addr` parsing logic.
    *   `auth.go`: Implements the core authorization router and interfaces.
    *   `common.go`: Provides Ethereum authentication structures and context extraction utilities.
    *   `common_test.go`: Tests address comparison functions.

**Launch Edge Cases:**

The package is a library and does not have a direct launch point. It's intended to be integrated into a gRPC server or client. The `AuthRouter` can be configured with different authorization implementations (e.g., allowing all requests, denying all requests, or verifying Ethereum addresses).

**Code Relations:**

*   `auth.go` and `common.go` work together to provide the core authorization logic. `auth.go` uses interfaces defined in `common.go` (e.g., `Authorization`) to implement different authorization strategies.
*   `addr.go` and `addr_test.go` provide address parsing and validation functionality, which is used by other parts of the package.
*   `common.go` extracts Ethereum addresses from gRPC contexts, which are then used by the `AuthRouter` to enforce authorization rules.

**Unclear/Dead Code:**

*   The `equalAddresses` function in `common.go` is marked as "TODO: Left for backward compabitility, prune later," indicating potential dead code.
*   The relationship between the `Addr` struct in `addr.go` and the rest of the package is not fully clear. It seems to be used for parsing addresses, but its integration with the authorization logic is not explicit.