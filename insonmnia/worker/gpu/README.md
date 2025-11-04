## GPU Package Summary

**Package Name:** `gpu`

This package manages GPU resources for containerized workloads, primarily focusing on NVIDIA and Radeon GPUs. It provides a vendor-agnostic interface for attaching GPUs to Docker containers, collecting metrics, and handling volume mounts. The package supports remote GPU tuners via gRPC, allowing for centralized GPU management.

**Configuration:**

*   **Environment Variables:** None explicitly defined in the provided code, but the `options.go` file suggests configuration via environment variables or external configuration files through `mapstructure.WeakDecode`.
*   **Flags/Cmdline Arguments:** Not directly present in the code, but the `remote_tuner.go` file implies a configurable `RemoteSocket` for gRPC communication.
*   **Files/Paths:**
    *   `/sys/dev/char/<major>:<minor>/device/drm/`: Used for DRI card detection.
    *   `/sys/class/drm/<card.Name>/device/vendor`: Reads GPU vendor ID.
    *   `/sys/class/drm/<card.Name>/device/uevent`: Reads PCI bus ID.
    *   `/dev/dri/`: Lists available DRI cards.
    *   `/sys/kernel/debug/dri/<card.Num>/amdgpu_pm_info`: Reads AMD GPU power consumption.
    *   Volume mount paths (configurable via `tunerOptions` in `options.go`).
*   **Edge Cases:**
    *   The `dri_windows.go` file returns an error on Windows, indicating no GPU support.
    *   The `tuner_other.go` file returns a `NilTuner` when GPU support is disabled via build tags (`!cl darwin`).
    *   Remote tuner functionality relies on a functional gRPC endpoint (`RemoteSocket`).

**Project Structure:**

```
insonmnia/worker/gpu/
├── detect.go
├── dri.go
├── dri_test.go
├── dri_unix.go
├── dri_windows.go
├── fake_tuner.go
├── interface.go
├── metrics.go
├── metrics_other.go
├── nvidia_tuner.go
├── options.go
├── radeon_tuner.go
├── remote_server.go
├── remote_tuner.go
├── tuner_other.go
├── utils.go
└── volume_plugin.go
```

**Relationships:**

*   `interface.go` defines the `Tuner` and `MetricsHandler` interfaces, which are implemented by vendor-specific tuners (`nvidia_tuner.go`, `radeon_tuner.go`, `fake_tuner.go`).
*   `metrics.go` and `metrics_other.go` handle GPU metric collection, with `metrics.go` focusing on NVIDIA and Radeon, while `metrics_other.go` provides a no-op implementation.
*   `volume_plugin.go` implements a Docker volume plugin for NVIDIA GPUs, relying on external volume management structures.
*   `remote_tuner.go` and `remote_server.go` enable remote GPU tuning via gRPC.
*   `utils.go` provides utility functions for container tuning, such as volume mount creation.

**Unclear Places/Dead Code:**

*   The `tuneContainer` function is referenced but not defined in the provided snippets, suggesting it may be implemented elsewhere.
*   The `collectDRICardsWithOpenCL` function in `radeon_tuner.go` is called but not defined, indicating missing implementation details.
*   The `Close` method in `radeon_tuner.go` is a no-op, potentially indicating incomplete functionality.
*   The `Unmount` method in `volume_plugin.go` simply returns an error, suggesting it may not be fully implemented.