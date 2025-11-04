# Package `gpu`

The **`gpu`** package implements a small GPU‑management subsystem that discovers AMD/RADEON and NVIDIA devices on the host, exposes them to Docker containers, and provides a gRPC service for remote tuning.  
At its core are three tuners (`fake`, `radeon`, `nvidia`) that all satisfy the same `Tuner` interface; each tuner knows how to discover devices, expose metrics, and bind them into a container.

---

## Short summary of provided files

| Path | Purpose |
|------|---------|
| `detect.go` | Helper functions for vendor lookup (`hasGPUWithVendor`, `GetVendorByName`). |
| `dri.go` | AMD‑specific device discovery (DRICard struct, constructor, metrics). |
| `dri_test.go` | Unit tests for the DRI regex and parsing helpers. |
| `dri_unix.go` / `dri_windows.go` | OS‑specific helper to read major/minor numbers from a sysfs path. |
| `fake_tuner.go` | Very small “dummy” tuner that creates fake devices (used as a fallback). |
| `interface.go` | Public API: `Tuner`, `MetricsHandler`, factory functions, and the nil‑stub implementation. |
| `metrics.go` / `metrics_other.go` | Concrete metric handlers for NVIDIA and Radeon; the latter is currently a stub that returns a `nilMetricsHandler`. |
| `nvidia_tuner.go` | Full NVIDIA tuner: discovers devices via OpenCL/NVML, builds a device map, logs it, and exposes tuning logic. |
| `options.go` | Configuration struct (`tunerOptions`) with functional options (`WithSocketDir`, `WithOptions`) and default constructors for each vendor. |
| `radeon_tuner.go` | Full Radeon tuner: matches DRI cards to OpenCL devices, builds a device map, logs it, and exposes tuning logic. |
| `remote_server.go` / `remote_tuner.go` | gRPC server that wraps a `Tuner`; the server implements `RemoteGPUTunerServer`. |
| `tuner_other.go` | Darwin‑specific fallback tuners that simply return a `NilTuner`. |
| `utils.go` | Helper to mount GPU devices into a Docker container (`newVolumeMount`, `tuneContainer`). |
| `volume_plugin.go` | Implements a Docker volume driver for NVIDIA volumes; exposes CRUD operations and a simple path helper. |

---

## Environment variables, flags & command‑line arguments

The package is configured through the following values:

| Variable / Flag | Description | Default / Source |
|-----------------|-------------|-------------------|
| `tunerOptions.VolumeDriverName` | Name of the driver (e.g., `"nvidia"` or `"radeon"`). | Set by `nvidiaDefaultOptions()` / `radeonDefaultOptions()`. |
| `tunerOptions.DriverVersion` | Driver version string. | Same as above. |
| `tunerOptions.VolumePath` | Base path where GPU volumes are stored on the host. | Default from options file. |
| `tunerOptions.DeviceCount` | Number of devices to create in a fake tuner. | Only used by `fake_tuner.go`. |
| `tunerOptions.RemoteSocket` | Address of the remote gRPC server (used by `remote_tuner.go`). | Set by `remoteDefaultOptions()`. |
| `tunerOptions.SocketPath` | Full Unix socket path for the gRPC service. | Built by `WithSocketDir()` using `path.Join`. |

Command‑line arguments are implicit:  
* The main entry point is a call to `New(ctx, vendorType, opts...)`, where `vendorType` is one of the enum values from `github.com/sonm-io/core/proto`.  
* The tuner type can be chosen at runtime by passing an appropriate option function (e.g., `WithOptions(map[string]string{…})`).  

---

## File structure

```
insonmnia/
└─ worker/
   └─ gpu/
      ├─ detect.go
      ├─ dri.go
      ├─ dri_test.go
      ├─ dri_unix.go
      ├─ dri_windows.go
      ├─ fake_tuner.go
      ├─ interface.go
      ├─ metrics.go
      ├─ metrics_other.go
      ├─ nvidia_tuner.go
      ├─ options.go
      ├─ radeon_tuner.go
      ├─ remote_server.go
      ├─ remote_tuner.go
      ├─ tuner_other.go
      ├─ utils.go
      └─ volume_plugin.go
```

---

## Relations between code entities

* **`New(ctx, vendorType, opts...)`** (in `interface.go`) dispatches to one of the concrete tuners (`fake`, `radeon`, `nvidia`).  
  * The chosen tuner builds a device map (`map[GPUID]*sonm.GPUDevice`) that is later used by `tuneContainer()` in `utils.go`.  
* **`DRICard`** (in `dri.go`) holds all low‑level information for an AMD card; its `Metrics()` method feeds into the Radeon metrics handler (`radeonMetrics`).  
  * The helper `collectRelatedDevices()` reads `/sys/dev/char/.../drm/`; `collectDeviceVendorIDs()` pulls vendor/device IDs from `/sys/class/drm/<card>/device/*`.  
* **`nvidiaMetrics`** (in `metrics.go`) wraps a slice of `nvidia.Device` objects; its `GetMetrics()` method aggregates temperature, fan speed and power into a map keyed by helper functions (`tempKey`, `fanKey`, `powerKey`).  
  * The same pattern is used for Radeon metrics.  
* **`remoteTunerService`** (in `remote_server.go`) holds a reference to the chosen tuner; its RPC method `Devices()` simply forwards to the tuner's `Devices()`.  
  * The client side (`remote_tuner.go`) creates a gRPC connection, fetches devices, and exposes them via `Tune()`.

---

## Edge cases for launching

1. **Local tuning** – Run `go run ./...` in the repository root; the main package will call `New(ctx, vendorType)` with default options (e.g., `nvidiaDefaultOptions()`).  
2. **Remote tuning** – Start the gRPC server by executing `remote_server.go`; it expects a running tuner instance and will expose a `RemoteGPUTunerServer`.  
3. **OS‑specific build tags** – The file `dri_unix.go` is compiled on non‑Windows platforms; `dri_windows.go` provides the Windows counterpart.  
4. **Darwin fallback** – If building for macOS without the `cl` tag, `tuner_other.go` supplies a minimal tuner that returns a `NilTuner`.  

---

## Unclear places / possible dead code

* The stub implementations in `metrics_other.go` and `tuner_other.go` currently return a `nilMetricsHandler`; if not overridden elsewhere they may be dead code.  
* In `options.go`, the field `libsMountPoint` is defined but never used; it might be intended for future volume mounting logic.  

---

## Summary of the whole package

The **gpu** package provides:

1. **Discovery** – AMD cards via DRI sysfs, NVIDIA devices via OpenCL/NVML.  
2. **Metrics collection** – per‑vendor handlers that expose fan speed, temperature and power.  
3. **Container binding** – `tuneContainer()` adds device mappings and volume mounts to a Docker host config.  
4. **Remote service** – a gRPC server that exposes the tuner’s devices for external consumption.  

All pieces are wired together through the public API in `interface.go`, making it easy to plug in new tuners or metrics handlers without touching the rest of the code base.