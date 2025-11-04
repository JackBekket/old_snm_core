## Package: `storage`

This package manages storage quotas, primarily focusing on Btrfs implementations. It provides interfaces for setting and cleaning up quotas, with specific implementations for Linux (Btrfs) and non-Linux platforms. The package interacts with Docker containers via the Docker API to apply quotas.

**Configuration:**

*   **Environment Variable:** `SUDO_USER` (used in tests to determine if sudo privileges are available).
*   **Docker Daemon:** The package relies on a running Docker daemon accessible via the `github.com/docker/docker/client` package.
*   **Image:** The "busybox" image is used for testing quota functionality.
*   **Hardcoded Limit:** A hardcoded quota limit of 20MB (`uint64(20 * 1024 * 1024)`) is used in tests.

**Launch Edgecases:**

*   The `btrfs_quota_test.go` test is skipped if the `SUDO_USER` environment variable is not set.
*   The `quotafactory_nonlinux.go` file is only compiled when the build tag `!linux` is active, meaning quota functionality is disabled on non-Linux platforms.

**Project Package Structure:**

```
insonmnia/worker/storage/
├── btrfs/
│   ├── btrfs_api.go
│   ├── btrfs_api_test.go
│   ├── btrfs_native_api.go
│   ├── btrfs_native_api_test.go
│   ├── cliapi_test.go
├── btrfs_quota_test.go
├── btrfs_quota_tuner.go
├── common.go
├── errors.go
├── quotafactory_linux.go
└── quotafactory_nonlinux.go
```

**Code Relations:**

*   `common.go` defines core interfaces (`QuotaDescription`, `Cleanup`, `StorageQuotaTuner`).
*   `errors.go` defines a custom error type for unsupported drivers.
*   `quotafactory_linux.go` and `quotafactory_nonlinux.go` provide platform-specific implementations for creating quota tuners.
*   `btrfs_quota_tuner.go` implements the Btrfs quota tuner, setting and cleaning up quotas using the Btrfs API.
*   `btrfs_quota_test.go` tests the Btrfs quota functionality using Docker containers.
*   The `btrfs` directory contains Btrfs-specific API implementations and tests.

**Unclear Places/Dead Code:**

*   The `btrfs_quota_tuner.go` file has a `TODO` comment indicating a missing rollback mechanism to prevent quota leaks.
*   The `quotafactory_nonlinux.go` file explicitly disables quota functionality on non-Linux platforms, suggesting that the package may have been designed with broader platform support in mind but currently only implements Btrfs quotas on Linux.