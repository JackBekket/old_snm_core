# Btrfs Package Summary

This package provides an API for managing Btrfs quotas, implemented using both command-line tool execution (`btrfsCLI`) and direct kernel interaction via `ioctl` system calls (`btrfsNativeAPI`). The package allows enabling, checking, creating, destroying, limiting, assigning, and removing quotas on Btrfs subvolumes.

**Configuration:**

*   **Environment Variables:**
    *   `BTRFS_PLAYGROUND_PATH`: Required for E2E tests, specifies the root path for testing.
    *   `SUDO_USER`: Indicates root permissions, required for E2E tests.
*   **Command-Line Arguments:**
    *   The `btrfsCLI` implementation relies on the `btrfs` command being available in the system's PATH.
*   **Files:**
    *   `btrfs_api.go`: Defines the `API` interface.
    *   `btrfs_api_test.go`: Tests the `API` interface.
    *   `btrfs_native_api.go`: Implements the `API` interface using `ioctl` system calls.
    *   `btrfs_native_api_test.go`: Tests the `btrfsNativeAPI` implementation.
    *   `cliapi_test.go`: Tests the `btrfsCLI` implementation.

**Launch Edgecases:**

*   The `btrfsCLI` implementation requires the `btrfs` command-line tool to be installed and in the system's PATH.
*   The `btrfsNativeAPI` implementation requires root privileges to execute `ioctl` system calls.
*   E2E tests in `btrfs_native_api_test.go` and `cliapi_test.go` are skipped if the `BTRFS_PLAYGROUND_PATH` and `SUDO_USER` environment variables are not set.

**Package Structure:**

```
insonmnia/worker/storage/btrfs/
├── btrfs_api.go
├── btrfs_api_test.go
├── btrfs_native_api.go
├── btrfs_native_api_test.go
└── cliapi_test.go
```

**Relations:**

*   `btrfs_api.go` defines the core `API` interface.
*   `btrfs_native_api.go` and `btrfsCLI` (in `btrfs_api.go`) both implement this interface.
*   The test files (`*_test.go`) verify the functionality of both implementations.
*   The `btrfsNativeAPI` implementation uses C bindings and `ioctl` system calls for direct kernel interaction.
*   The `btrfsCLI` implementation wraps the `btrfs` command-line tool.

**Unclear Places/Dead Code:**

*   The `testE2EOne` function is referenced in multiple test files but not defined in the provided code snippets.
*   The `parseQgroupID` function is tested in `btrfs_native_api_test.go` but not defined in the provided code snippets.
*   The exact implementation details of the `btrfsCLI` and `btrfsNativeAPI` are not fully visible without the complete source code.