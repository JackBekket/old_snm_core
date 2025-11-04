# sysinit

## Overview  
`sysinit` is a small Go library that wraps the Linux `lsblk` utility and orchestrates a set of actions to create an encrypted Docker volume, mount it, and restart Docker.  The package exposes two public functions:

* **`ListBlockDevices(ctx context.Context)`** – runs `lsblk --output NAME,FSTYPE --json`, decodes the JSON into a slice of `BlockDevice` structs, and returns that list.
* **`NewInitService(cfg Config, logger *zap.Logger) (*InitService, error)`** – creates an instance that can reset or mount the system.  The service builds a queue of actions (cryptsetup, mkfs, Docker restart, etc.) and executes them in order.

The code is organized into three files:

```
insonmnia/
├── sysinit/
│   ├── lsblk.go
│   ├── service.go
│   └── service_test.go
```

---

## Environment variables / flags / command‑line arguments

| Source | Variable / flag | Description |
|--------|-----------------|-------------|
| `lsblk.go` | `--output NAME,FSTYPE --json` | Flags passed to the external `lsblk` binary.  They request only the device name and filesystem type in JSON format. |
| `service.go` | `Config` fields (`device`, `cipher`, `fs_type`, `mount_point`) | These are read from a YAML configuration file (tags `yaml:"..."`).  The values drive the cryptsetup, mkfs, and mount commands. |
| `service.go` | `exec.CommandContext(ctx, "lsblk", …)` | Context‑aware execution of external commands. |
| `service.go` | `time.Duration(10 * time.Second)` | Timeout for the action queue in `Mount`. |

---

## File structure

```
insonmnia/sysinit/
├── lsblk.go          – BlockDevice type & ListBlockDevices helper
├── service.go        – InitService, Config, actions, and orchestration logic
└── service_test.go   – unit tests for CreateFileSystemAction.findDevice
```

---

## Key code entities and their relationships

| Entity | Purpose | Related code |
|--------|---------|--------------|
| `BlockDevice` (lsblk.go) | Represents a block device returned by `lsblk`. | Used in `ListBlockDevices`, passed to actions that need a device name. |
| `Config` (service.go) | Holds runtime parameters for the service. | Injected into `InitService`; fields are referenced by all action types. |
| `InitService` (service.go) | Orchestrates the action queue. | Calls `makeActions()` → executes actions in order. |
| `CreateEncryptedVolumeAction`, `CreateMountPointAction`, `CreateFileSystemAction`, `MountDeviceMapperAction`, `StartDockerAction`, `FailAction` | Individual steps of the service. | Each has an `Execute(ctx)` method that performs a system call; they are chained by `makeActions()`. |
| `action.findDevice(devices []*BlockDevice) (*BlockDevice, error)` (in CreateFileSystemAction) | Recursively searches for a device by name. | Tested in `service_test.go`; used to locate the target mapper before running `mkfs`. |

The actions are executed sequentially:

1. **CreateEncryptedVolumeAction** – runs `cryptsetup create …`.
2. **CreateMountPointAction** – creates the mount directory.
3. **CreateFileSystemAction** – finds the device, then runs `mkfs.<type>` on `/dev/mapper/<name>`.
4. **MountDeviceMapperAction** – mounts the mapper to the mount point via Docker’s mount helper.
5. **StartDockerAction** – restarts `docker.service` through DBus.
6. **FailAction** – a dummy action that always fails (used for testing).

---

## Edge cases & launch scenarios

* **Running tests** – `go test ./...` will compile all three files and run the two unit tests in `service_test.go`.  The tests exercise the recursive search logic of `CreateFileSystemAction.findDevice`.
* **Executing the service** – A main package could instantiate a logger, load a YAML config into `Config`, then call `NewInitService(cfg, logger)` followed by `svc.Mount(ctx)`.  Because `Mount` checks `isDockerRunning()` before executing actions, it can be invoked repeatedly to bring Docker back into a known state.
* **Environment variables** – The package currently does not read any env vars directly; all configuration comes from the YAML tags in `Config`.

---

## Summary of logic

1. **lsblk.go**  
   * Defines `BlockDevice` and provides `ListBlockDevices(ctx)` that runs `lsblk`, decodes JSON, and returns a slice of devices.

2. **service.go**  
   * Holds configuration (`Config`) and the orchestrator (`InitService`).  
   * Builds an action queue in `makeActions()`.  
   * Each action performs a system call (cryptsetup, mkfs, mount helper, DBus restart).  
   * The service can be reset or mounted; both paths execute the same queue.

3. **service_test.go**  
   * Provides two unit tests that confirm `CreateFileSystemAction.findDevice` correctly finds a device either at the top level or nested under another device.

The package is ready to be used as part of a larger system‑initialization workflow, or as a standalone CLI if wrapped in a `main` package.