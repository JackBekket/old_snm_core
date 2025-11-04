<br/>

```markdown
## Package: `plugin`

This package manages SONM plugins for Docker containers, handling GPU, volume, network, and storage quota tuning based on provider interfaces and configuration settings. It initializes, configures, and cleans up these plugins within a worker environment.

**Imports:**

*   `context`, `fmt`, `sort`: Standard Go packages for context management, formatted output, and sorting.
*   `github.com/docker/docker/...`: Docker client API types for container, network configurations.
*   `github.com/noxiouz/zapctx/ctxlog`, `go.uber.org/zap`: Structured logging with context awareness using Zap.
*   `github.com/sonm-io/core/insonmnia/...`: Custom data structures and worker-specific functionality (GPU, network, storage, volume).
*   `github.com/sonm-io/core/proto`: Protocol definitions for SONM communication.

**Configuration:**

The package relies on a `Config` struct loaded from external sources (e.g., YAML) to define plugin settings:

*   `SocketDir`: Path to plugin sockets (`/run/docker/plugins` by default).
*   `Volumes`: Root path for Docker volumes (`/var/lib/docker-volumes` by default) and driver configurations.
*   `Overlay`: Tinc and L2TP network configurations.

**Interfaces:**

The package uses provider interfaces (e.g., `GPUProvider`, `VolumeProvider`) to abstract plugin interactions, enabling modularity. These interfaces define methods for tuning specific container resources.

**Key Functions:**

*   `NewRepository()`: Initializes the repository by loading plugins based on a configuration file.
*   `Tune()`: Orchestrates the tuning process using provider interfaces (GPU, volumes, network, storage quota). Returns a `Cleanup` function to revert changes if needed.

**Volume Tuning (`TuneVolumes()`):** Creates and configures Docker volumes based on volume specifications from the `VolumeProvider`. Handles cleanup of created volumes.

**GPU Tuning (`TuneGPU()`):** Applies GPU settings to the container's host configuration using registered GPU tuners.

**Network Tuning (`TuneNetworks()`):** Configures network settings for containers, leveraging network tuners based on provider specifications.

**Storage Quota Tuning (`TuneStorageQuota()`):** Sets storage quotas for containers if supported by the Docker driver and enabled in the configuration.

**Cleanup:** The `Cleanup` interface ensures that resources are released when a container is stopped or removed.  The nested cleanup mechanism handles multiple cleanup operations sequentially, aggregating errors. Volume cleanup uses the `VolumeDriver` to remove volumes.

**File Structure:**

*   `cleanup.go`: Defines interfaces and structures for resource cleanup (nested cleanup, volume removal).
*   `config.go`: Defines configuration structs for socket directory, volumes, and overlay networks.
*   `plugin.go`: Implements the main repository logic for managing plugins, tuning containers, and handling cleanup.

**Edge Cases:**

The package assumes a valid Docker client is available. Errors during plugin initialization or tuning are handled by logging and returning errors to the caller. The behavior after calling `Close()` on nested cleanups multiple times is undefined.
```