## btrfs

This package provides an API for managing BTRFS quotas using both native system calls (via C bindings in `btrfs_native_api.go`) and CLI-based execution (`btrfs_api.go`). The core functionality revolves around enabling, creating, destroying, limiting, assigning, removing, checking existence, and retrieving IDs of BTRFS quota groups.

**Configuration:**
*   `BTRFS_PLAYGROUND_PATH`: Environment variable specifying the root path for E2E tests (required by `btrfs_native_api_test.go`, `cliapi_test.go`). Tests skip if not set.
*   `SUDO_USER`: Environment variable indicating whether the test is running with root privileges (required by `btrfs_native_api_test.go`, `cliapi_test.go`). Tests skip if not set.

**Files:**
*   `btrfs_api.go`: Implements the API interface using CLI calls to `btrfs`.
*   `btrfs_api_test.go`: E2E tests for the CLI-based API implementation.
*   `btrfs_native_api.go`: Implements the API interface using native BTRFS IOCTL system calls (C bindings).
*   `btrfs_native_api_test.go`: E2E tests for the native API implementation.
*   `cliapi_test.go`: Tests helper functions used in CLI-based operations.

**Edge Cases:**
The `btrfsCLI` implementation relies on the external `btrfs` executable being present in the system's PATH. The native API (`btrfs_native_api.go`) requires root privileges to execute IOCTL commands, enforced by environment variable checks in tests. Tests skip if required env vars are not set or sudo access is missing.

**Relations:**
The `API` interface defines a common contract for both CLI-based and native implementations. The `btrfsCLI` struct wraps the external `btrfs` command, while `btrfsNativeAPI` directly interacts with the kernel via IOCTLs. Tests in `*_test.go` files verify the correctness of these implementations under various conditions.