# insonmnia/worker/storage/btrfs_quota_test.go  
## Package: `storage`  
  
**Imports:**  
  
*   `context`  
*   `os`  
*   `strings`  
*   `testing`  
*   `github.com/docker/docker/api/types`  
*   `github.com/docker/docker/api/types/container`  
*   `github.com/docker/docker/api/types/network`  
*   `github.com/docker/docker/client`  
*   `github.com/stretchr/testify/require`  
  
**External Data/Input Sources:**  
  
*   Environment variable `SUDO_USER` (used for skipping the test if not set).  
*   Docker daemon (accessed via `github.com/docker/docker/client`).  
*   Busybox image ("busybox") is used for container creation.  
*   Hardcoded limit value: `uint64(20 * 1024 * 1024)` (20MB).  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
*   **TestBTRFSQuota Function:** This function tests BTRFS quota functionality using the Docker API. It creates three containers based on the "busybox" image, sets a quota limit of 20MB for each container, and then attempts to write data to a file within each container. The test verifies that one container fails to write due to the quota limit, while at least one container successfully writes some data (less than the limit).  
*   **Container Management:** The code creates and removes Docker containers using the Docker client. It uses a `defer` statement to ensure that all containers are removed and quota settings are cleaned up, even if the test fails.  
*   **Quota Setting:** The `NewQuotaTuner` function (not fully shown in the snippet) is used to create a quota tuner, which is then used to set the quota for each container. The `SetQuota` method is called to apply the quota.  
*   **Data Verification:** The code attempts to copy data from each container's `/FILE` to verify the amount of data written. It checks that one container fails (due to the quota) and that at least one container writes some data within the limit.  
*   **Conditional Test Execution:** The test is skipped if the `SUDO_USER` environment variable is not set, indicating that sudo privileges are required.  
  
# insonmnia/worker/storage/btrfs_quota_tuner.go  
## Package: `storage`  
  
**Imports:**  
  
*   `context`  
*   `fmt`  
*   `hash/fnv`  
*   `io`  
*   `io/ioutil`  
*   `path/filepath`  
*   `github.com/docker/docker/api/types`  
*   `github.com/sonm-io/core/insonmnia/worker/storage/btrfs`  
  
**External Data/Input Sources:**  
  
*   `types.Info` struct (from `github.com/docker/docker/api/types`) containing `Driver` and `DockerRootDir` fields.  
*   File system access to read `mount-id` from `info.DockerRootDir/image/btrfs/layerdb/mounts/{ID}/mount-id`.  
*   Btrfs API via `github.com/sonm-io/core/insonmnia/worker/storage/btrfs`.  
  
**TODOs:**  
  
*   `// TODO: add ROLLBACK to prevent quota leak` in `SetQuota` function.  
  
**Code Summary:**  
  
### Btrfs Quota Management  
  
This code implements Btrfs quota management for Docker containers. It provides two main structures: `btrfsQuotaTuner` and `btrfsQuotaCleaner`.  
  
*   `btrfsQuotaTuner`: Responsible for setting quotas on Btrfs subvolumes. It takes a `types.Info` struct to determine if the driver is Btrfs. It initializes a Btrfs API and sets up the quota using the container ID and desired quota size. It creates a qgroup if it doesn't exist, limits it, and assigns the container to the qgroup.  
*   `btrfsQuotaCleaner`: Responsible for cleaning up quotas. It removes the quota assignment and destroys the qgroup. It includes a comment noting that the qgroup won't be removed if subvolumes are still assigned to it.  
  
### `SetQuota` Function  
  
The `SetQuota` function is the core logic for setting quotas. It reads the mount ID from a file, enables quotas, creates a qgroup (if needed), limits the qgroup, assigns the container to the qgroup, and returns a `btrfsQuotaCleaner` for cleanup. The function uses a hash of the `quotaID` to generate the qgroupID.  
  
### `newBtrfsQuotaTuner` Function  
  
The `newBtrfsQuotaTuner` function creates a new `btrfsQuotaTuner` instance. It checks if the driver is Btrfs and returns an error if it's not. It initializes the Btrfs API and sets the Docker root directory and subvolumes directory.  
  
# insonmnia/worker/storage/common.go  
## Package: `storage`  
  
**Imports:**  
  
*   `context`  
  
**External Data/Input Sources:**  
  
*   None explicitly defined in this snippet. The package relies on external context for operations.  
  
**TODOs:**  
  
*   None found in this snippet.  
  
---  
  
### Core Definitions  
  
This package defines core interfaces and structures related to storage quotas. `QuotaDescription` holds the quota size in bytes. `Cleanup` is an interface for resources that need to be closed (e.g., quota handles). `StorageQuotaTuner` is the primary interface for setting storage quotas, taking a context, ID, quota ID, and byte size as input, and returning a `Cleanup` object for resource management.  
  
---  
  
This snippet provides the basic building blocks for managing storage quotas within a larger system. It focuses on defining interfaces for setting and cleaning up quotas, leaving the actual implementation details to other parts of the package or external components.  
  
# insonmnia/worker/storage/errors.go  
## Package: `storage`  
  
**Imports:**  
  
*   `fmt`  
  
**External Data/Input Sources:**  
  
*   None. This code defines a custom error type and its string representation. It does not interact with external data sources or take any input.  
  
**TODOs:**  
  
*   None.  
  
**Code Summary:**  
  
### Custom Error Definition  
  
The code defines a custom error type `ErrDriverNotSupported` which embeds a string representing the unsupported driver. The `Error()` method implements the `error` interface, returning a formatted string indicating which driver is not supported. This error type is likely used to signal that a requested storage driver is not implemented or available within the `storage` package.  
  
# insonmnia/worker/storage/quotafactory_linux.go  
## Package: `storage`  
  
**Imports:**  
  
*   `github.com/docker/docker/api/types`  
  
**External Data/Input Sources:**  
  
*   `types.Info` struct from `github.com/docker/docker/api/types` is used as input to `NewQuotaTuner`. Specifically, the `Driver` field of this struct determines the quota tuner to be created.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
**Summary of Major Code Parts:**  
  
*   **Platform Support:** The `PlatformSupportsQuota` variable is set to `true`, indicating that the current platform supports storage quotas.  
*   **Quota Tuner Creation:** The `NewQuotaTuner` function creates a storage quota tuner based on the provided `types.Info` struct. It currently supports only the "btrfs" driver. If the driver is not "btrfs", it returns an error indicating that the driver is not supported.  
  
# insonmnia/worker/storage/quotafactory_nonlinux.go  
## Package: `storage`  
  
**Imports:**  
  
*   `fmt`  
*   `runtime`  
*   `github.com/docker/docker/api/types`  
  
**External Data/Input Sources:**  
  
*   `types.Info` (from `github.com/docker/docker/api/types`) - Used as input to `NewQuotaTuner`.  
*   `runtime.GOOS` - Used to determine the operating system.  
  
**TODOs:**  
  
*   None.  
  
**Summary of Major Code Parts:**  
  
*   **Platform Support:** The `PlatformSupportsQuota` variable is set to `false`, indicating that quota functionality is not supported on the current platform.  
*   **Quota Tuner Creation:** The `NewQuotaTuner` function always returns an error, stating that quota is not supported on the current platform (determined by `runtime.GOOS`). It returns `nil` for the `StorageQuotaTuner` interface. This function is only compiled when the build tag `!linux` is active.  
  
