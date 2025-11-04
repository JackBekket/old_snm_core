# insonmnia/worker/gpu/detect.go  
## GPU Package Component Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `errors`: For error handling.  
*   `strings`: For string manipulation (specifically, converting vendor names to uppercase).  
*   `github.com/sonm-io/core/proto`: For `sonm.GPUVendorType` and `sonm.GPUDevice` types.  
  
**External Data/Input Sources:**  
*   `sonm.GPUDevice` slice: Input to `hasGPUWithVendor` function, representing the available GPU devices on the system.  
*   `vendor` string: Input to `GetVendorByName` function, representing the GPU vendor name.  
  
**TODOs:**  
*   None found in this code snippet.  
  
**Code Part Summaries:**  
  
### `hasGPUWithVendor` Function  
This function checks if a GPU with a specific vendor type (`sonm.GPUVendorType`) exists within a slice of `sonm.GPUDevice` objects. It iterates through the devices and returns an error if no matching vendor is found.  
  
### `GetVendorByName` Function  
This function converts a GPU vendor name (string) to its corresponding `sonm.GPUVendorType` enum value. It converts the input string to uppercase and uses the `sonm.GPUVendorType_value` map to perform the lookup. If the vendor name is unknown, it returns `sonm.GPUVendorType_GPU_UNKNOWN` and an error.  
  
# insonmnia/worker/gpu/dri.go  
## GPU Package - `gpu` Component Summary  
  
**Package/Component Name:** `gpu`  
  
**Imports:**  
  
*   `bufio`: For buffered I/O operations (reading files line by line).  
*   `errors`: For creating custom error types.  
*   `fmt`: For formatted I/O (printing, string formatting).  
*   `io/ioutil`: For file reading and directory listing.  
*   `os`: For operating system interactions (file opening, closing).  
*   `path`: For path manipulation (joining paths).  
*   `regexp`: For regular expression matching.  
*   `strconv`: For string to number conversions.  
*   `strings`: For string manipulation.  
  
**External Data/Input Sources:**  
  
*   `/sys/dev/char/<major>:<minor>/device/drm/`: Used to find related DRI devices.  
*   `/sys/class/drm/<card.Name>/device/vendor`: Reads the vendor ID.  
*   `/sys/class/drm/<card.Name>/device/device`: Reads the device ID.  
*   `/sys/class/drm/<card.Name>/device/uevent`: Reads PCI bus ID.  
*   `/sys/dev/char/<major>:<minor>/device/hwmon`: Reads hardware monitoring data (temperature, fan speed).  
*   `/sys/kernel/debug/dri/<card.Num>/amdgpu_pm_info`: Reads power consumption data (AMD GPUs).  
*   `/dev/dri/`: Lists available DRI cards.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### DRICard Struct and Initialization  
  
The `DRICard` struct represents a DRI card with attributes like number, name, path, devices, IDs, bus ID, memory, and hwmon path. The `NewDRICard` function initializes a `DRICard` instance, collecting related devices, vendor IDs, and PCI bus ID.  
  
### Device Collection Methods  
  
*   `collectRelatedDevices`: Queries `/sys/dev/char` to find related DRI devices based on major and minor numbers.  
*   `collectDeviceVendorIDs`: Reads vendor and device IDs from `/sys/class/drm`.  
*   `collectPCIBusID`: Parses the PCI slot name from `/sys/class/drm/<card.Name>/device/uevent`.  
*   `collectHwmonPath`: Finds the hardware monitoring path in `/sys/dev/char/<major>:<minor>/device/hwmon`.  
  
### Metrics Reading Methods  
  
*   `readFanSpeed`: Reads PWM values from `/sys/class/drm/<card.Name>/hwmon` to calculate fan speed.  
*   `readTemperature`: Reads temperature from `/sys/class/drm/<card.Name>/hwmon`.  
*   `readPowerConsumption`: Reads power consumption from `/sys/kernel/debug/dri/<card.Num>/amdgpu_pm_info` (AMD GPUs).  
  
### Utility Functions  
  
*   `readSysClassValue`: Reads a value from `/sys/class` and parses it as a uint64.  
*   `parseSysClassValue`: Parses a byte slice as a uint64 in hexadecimal format.  
*   `parsePCISlotName`: Extracts the PCI slot name from a uevent string.  
  
### Card Discovery Functions  
  
*   `CollectDRICardDevices`: Lists DRI cards in `/dev/dri` and creates `DRICard` instances.  
*   `newCardDeviceByName`: Creates a `DRICard` from a card name (e.g., "card0").  
*   `newCardByDevicePath`: Creates a `DRICard` from a device path (e.g., "/dev/nvidia1").  
  
# insonmnia/worker/gpu/dri_test.go  
## GPU Package - Test File Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `fmt` (for formatted I/O)  
*   `strconv` (for string conversion)  
*   `testing` (for unit tests)  
*   `github.com/stretchr/testify/assert` (for assertions in tests)  
  
**External Data/Input Sources:**  
*   Test cases defined as structs within each test function. These tests provide input strings and expected outputs for various parsing and matching functions.  
*   Regular expression `devDRICardNameRe` (not defined in this snippet, assumed to be defined elsewhere in the package) used for matching device names.  
*   Byte slices created from input strings for `parseSysClassValue`.  
  
**TODOs:**  
*   None found in this code snippet.  
  
**Code Summaries:**  
  
### `TestMatchDeviceName`  
This test function verifies the correct parsing of device names using a regular expression (`devDRICardNameRe`). It checks if the device name matches the expected format (e.g., "card0", "card2") and if the extracted number matches the input number.  The tests cover both matching and non-matching cases, including invalid formats.  
  
### `TestParseSysClassValue`  
This test function tests the `parseSysClassValue` function (not defined in this snippet, assumed to be defined elsewhere in the package) which parses hexadecimal or decimal values from a byte slice. It checks for valid hexadecimal and decimal inputs (e.g., "0x0", "fff") and ensures that invalid inputs (e.g., "", "\r", "0xp1d0r") result in an error.  
  
### `TestParsePCISlotName`  
This test function tests the `parsePCISlotName` function (not defined in this snippet, assumed to be defined elsewhere in the package) which parses PCI slot names from strings. It verifies that valid PCI slot names (e.g., "PCI\_SLOT\_NAME=0000:01:00.0") are correctly extracted, while invalid inputs (e.g., "", "aaabbb") return an empty string and `false`.  
  
# insonmnia/worker/gpu/dri_unix.go  
## Package: `gpu`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O, specifically error formatting.  
*   `syscall`: For making low-level system calls, including file stat operations.  
  
**External Data/Input Sources:**  
  
*   `path` (string): The file path to a device node (e.g., `/dev/nvidia0`). This is the primary external input.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
**Summary of Major Code Parts:**  
  
### `deviceNumber` Function  
  
This function extracts the major and minor device numbers from a given device node path. It uses `syscall.Stat` to retrieve file statistics, then calculates the major and minor numbers from the `Rdev` field of the `Stat_t` struct. If `syscall.Stat` fails, it returns an error. The function is specifically built for non-Windows systems (indicated by the `// +build !windows` directive).  
  
# insonmnia/worker/gpu/dri_windows.go  
## GPU Package (Windows Build)  
  
**Package Name:** `gpu`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O, specifically error creation.  
*   `runtime`: For accessing runtime environment information (GOOS).  
  
**External Data/Input Sources:**  
  
*   `path` (string): Input parameter to the `deviceNumber` function, though it's unused in the current implementation.  
*   `runtime.GOOS`: Used to determine the operating system.  
  
**TODOs:**  
  
*   None.  
  
**Code Summary:**  
  
### `deviceNumber` Function  
  
This function is specifically compiled only for Windows builds (due to the `// +build windows` directive). It currently returns an error indicating that the platform is unsupported. The function takes a `path` string as input, but it is not used. The function always returns 0, 0, and an error message indicating that the current operating system is not supported. This suggests that the GPU functionality is not implemented for non-Windows platforms.  
  
# insonmnia/worker/gpu/fake_tuner.go  
## GPU Package Component Summary: `fake_gpu_tuner.go`  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `context`  
*   `fmt`  
*   `github.com/docker/docker/api/types/container`  
*   `github.com/noxiouz/zapctx/ctxlog`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `go.uber.org/zap`  
  
**External Data/Input Sources:**  
*   `container.HostConfig`: Docker container host configuration.  
*   `[]GPUID`: Slice of GPU IDs.  
*   `context.Context`: Used for logging.  
*   `options`: Configuration options passed to `newFakeTuner`.  
  
**TODOs:**  
*   None found in this file.  
  
**Code Summary:**  
  
### `fakeGPUTuner` Struct  
This struct represents a fake GPU tuner, used for testing or environments where real GPU access is not required. It holds a logger and a slice of fake `GPUDevice` structs.  
  
### `newFakeTuner` Function  
This function creates a new instance of `fakeGPUTuner`. It initializes a configurable number of fake GPU devices with predefined properties (ID, Vendor, Memory, etc.). The number of devices is determined by the `DeviceCount` option. The function returns the tuner instance and nil error.  
  
### `Tune` Method  
This method simulates GPU tuning for a Docker container. It logs a debug message indicating the tuning process with the provided device IDs but does not perform any actual tuning. It always returns nil error.  
  
### `Devices` Method  
This method returns the list of fake GPU devices managed by the tuner. It logs a debug message before returning the devices.  
  
### `Close` Method  
This method simulates closing the fake GPU driver. It logs a debug message and returns nil error.  
  
# insonmnia/worker/gpu/interface.go  
## GPU Package Component Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `context`  
*   `github.com/docker/docker/api/types/container`  
*   `github.com/noxiouz/zapctx/ctxlog` (aliased as `log`)  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `go.uber.org/zap`  
  
**External Data/Input Sources:**  
*   `sonm.GPUVendorType`: Enum defining GPU vendor types (RADEON, NVIDIA, FAKE, REMOTE). Used as input to `New` and `NewMetricsHandler`.  
*   `container.HostConfig`: Docker container host configuration, modified by `Tuner.Tune`.  
*   `GPUID`: String type representing GPU identifiers. Input to `Tuner.Tune`.  
  
**TODOs:**  
*   No TODO comments found in this file.  
  
---  
  
### Tuner Interface & Creation  
  
The `Tuner` interface is central to GPU management. It provides methods to attach GPUs to containers (`Tune`), list available devices (`Devices`), and clean up resources (`Close`). The `New` function creates a `Tuner` implementation based on the provided `GPUVendorType`. Currently supports Radeon, NVIDIA, FAKE, and REMOTE tuners. If the type is unrecognized, a `NilTuner` is returned.  
  
### Metrics Handling  
  
The `MetricsHandler` interface provides a way to retrieve GPU metrics (`GetMetrics`) and release resources (`Close`). The `NewMetricsHandler` function creates a metrics handler based on the `GPUVendorType`.  A `nilMetricsHandler` is returned for unsupported types.  
  
### Nil Implementations  
  
`NilTuner` and `nilMetricsHandler` are null implementations of the `Tuner` and `MetricsHandler` interfaces, respectively. They provide no-op behavior and are used when a GPU type is not recognized or when no actual GPU functionality is needed.  
  
---  
  
This component focuses on abstracting GPU management and metrics collection, providing a vendor-agnostic interface for interacting with different GPU types. The use of interfaces and factory functions (`New`, `NewMetricsHandler`) promotes flexibility and extensibility. The `NilTuner` and `nilMetricsHandler` provide graceful fallback mechanisms for unsupported configurations.  
  
# insonmnia/worker/gpu/metrics.go  
## GPU Metrics Handler Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `fmt` (for formatted I/O)  
*   `sync` (for synchronization primitives like mutexes)  
*   `github.com/sonm-io/core/proto` (likely for metric key definitions)  
*   `github.com/sshaman1101/nvidia-docker/nvidia` (for NVIDIA GPU interaction via NVML)  
  
**External Data/Input Sources:**  
*   NVIDIA GPUs via NVML (NVIDIA Management Library)  
*   DRI cards (AMD GPUs) via OpenCL (through `collectDRICardsWithOpenCL` function, not shown in the provided snippet)  
*   `sonm.MetricsKeyGPUPrefix`, `sonm.MetricsKeyGPUTemperature`, `sonm.MetricsKeyGPUFan`, `sonm.MetricsKeyGPUPower` (constants from `github.com/sonm-io/core/proto` used for metric key formatting)  
  
**TODOs:**  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### NVIDIA Metrics Handling  
The `nvidiaMetrics` struct and associated functions (`newNvidiaMetricsHandler`, `GetMetrics`, `Close`) handle collecting metrics from NVIDIA GPUs.  `newNvidiaMetricsHandler` initializes NVML, looks up available NVIDIA devices, and returns a `nvidiaMetrics` handler. `GetMetrics` retrieves temperature, fan speed, and power usage for each detected NVIDIA GPU using NVML.  The `Close` method shuts down NVML.  Error handling is present for NVML initialization, device lookup, and status retrieval.  
  
### Radeon Metrics Handling  
The `radeonMetrics` struct and associated functions (`newRadeonMetricsHandler`, `GetMetrics`, `Close`) handle collecting metrics from AMD GPUs (DRI cards). `newRadeonMetricsHandler` calls `collectDRICardsWithOpenCL` (implementation not provided) to find AMD GPUs and returns a `radeonMetrics` handler. `GetMetrics` retrieves temperature, fan speed, and power usage for each detected AMD GPU. The `Close` method does nothing. Error handling is present for device status retrieval.  
  
### Metric Key Generation  
The `tempKey`, `fanKey`, and `powerKey` functions generate unique metric keys using a prefix, GPU index, and metric type (temperature, fan speed, power). These keys are used to store metrics in a map. The prefix is taken from the `sonm.MetricsKeyGPUPrefix` constant.  
  
# insonmnia/worker/gpu/metrics_other.go  
## GPU Package - File Summary  
  
**Package Name:** `gpu`  
  
**Imports:** None  
  
**External Data/Input Sources:** None. This file does not interact with any external data sources or take any input.  
  
**TODOs:** None.  
  
### Code Summary  
  
This file defines functions to create metrics handlers for Nvidia and Radeon GPUs. However, both functions currently return a `nilMetricsHandler` and no error, effectively providing no functionality. The `+build !cl darwin` directive indicates that this code is excluded when building for the `cl` (CUDA) toolchain or on Darwin (macOS). This suggests the file is intended for non-CUDA/non-macOS GPU metric handling, but currently does nothing.  
  
# insonmnia/worker/gpu/nvidia_tuner.go  
## GPU Package Component Summary: `gpu`  
  
**Package Name:** `gpu`  
  
**Imports:**  
  
*   `context`  
*   `fmt`  
*   `net`  
*   `os`  
*   `sync`  
*   `syscall`  
*   `github.com/docker/docker/api/types/container`  
*   `github.com/docker/go-connections/sockets`  
*   `github.com/docker/go-plugins-helpers/volume`  
*   `github.com/noxiouz/zapctx/ctxlog` (aliased as `log`)  
*   `github.com/sonm-io/core/insonmnia/hardware/gpu` (aliased as `sonm`)  
*   `github.com/sonm-io/core/proto`  
*   `github.com/sshaman1101/nvidia-docker/nvidia`  
*   `go.uber.org/zap`  
  
**External Data/Input Sources:**  
  
*   OpenCL GPU devices (obtained via `gpu.GetGPUDevices()`)  
*   NVIDIA driver version (obtained via `nvidia.GetDriverVersion()`)  
*   NVIDIA management library (NVML) initialization and shutdown.  
*   Control device paths (obtained via `nvidia.GetControlDevicePaths()`)  
*   Volume information (defined in `volInfo` and passed to `nvidia.LookupVolumes()`)  
*   Docker socket path (configurable via `tunerOptions`)  
*   Volume path (configurable via `tunerOptions`)  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### `nvidiaTuner` Struct  
  
The `nvidiaTuner` struct manages NVIDIA GPU devices for container tuning. It holds tuner options, a volume handler, a listener for Docker communication, a mutex for synchronization, and a map of GPU IDs to `sonm.GPUDevice` objects.  
  
### `Tune` Method  
  
The `Tune` method locks the tuner's mutex, then calls the `tuneContainer` function (not shown in this snippet) to apply GPU configuration to a Docker container's host configuration.  
  
### `Devices` Method  
  
The `Devices` method locks the tuner's mutex and returns a slice of `sonm.GPUDevice` objects from the `devMap`.  
  
### `Close` Method  
  
The `Close` method closes the listener socket and removes the socket file.  
  
### `newNvidiaTuner` Function  
  
This function initializes the `nvidiaTuner`. It performs the following steps:  
  
1.  Loads NVIDIA unified memory (UVM) and initializes NVML.  
2.  Retrieves GPU devices using `nvidia.LookupDevices()`.  
3.  Creates `sonm.GPUDevice` objects for each detected NVIDIA GPU, populating their fields with device information (ID, vendor, name, memory, etc.).  
4.  Sets up volume handling using `nvidia.LookupVolumes()` and `volume.NewHandler()`.  
5.  Creates a Unix socket for Docker plugin communication using `sockets.NewUnixSocket()`.  
6.  Starts a goroutine to serve Docker requests via the volume handler.  
  
The function returns a pointer to the initialized `nvidiaTuner` or an error if any step fails.  
  
# insonmnia/worker/gpu/options.go  
## GPU Package Component Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `fmt` (for formatted I/O)  
*   `path` (for path manipulation)  
*   `github.com/mitchellh/mapstructure` (for decoding maps into structs)  
  
**External Data/Input Sources:**  
*   Configuration maps (`map[string]string`) passed to `WithOptions` for dynamic option setting.  
*   Environment variables or external configuration files (implied usage through `mapstructure` decoding).  
*   Hardcoded constants for driver names, versions, and mount points (nvidia, radeon).  
*   Directory path for socket creation (passed to `WithSocketDir`).  
  
**TODOs:**  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Constants and Data Structures  
The code defines several constants related to NVIDIA and Radeon GPU drivers, including driver names, versions, and mount points. The `tunerOptions` struct holds configuration parameters for GPU tuners, including driver name, version, volume path, device count, and socket paths. The struct uses `mapstructure` tags for easy decoding from maps.  
  
### Option Functions  
The `Option` type is a function that modifies a `tunerOptions` struct. `WithSocketDir` sets the socket path based on the driver name and a provided directory. `WithOptions` decodes a map of strings into the `tunerOptions` struct using `mapstructure.WeakDecode`.  
  
### Default Options Functions  
Functions like `nvidiaDefaultOptions`, `radeonDefaultOptions`, `fakeDefaultOptions`, and `remoteDefaultOptions` return pre-configured `tunerOptions` structs with default values for different scenarios (NVIDIA, Radeon, fake, remote). These functions initialize the `tunerOptions` struct with hardcoded values.  
  
### Volume Name Generation  
The `volumeName` method on the `tunerOptions` struct generates a volume name based on the driver name and version.  
  
# insonmnia/worker/gpu/radeon_tuner.go  
Package Name: `gpu`  
  
Imports:  
- `context`  
- `errors`  
- `sync`  
- `github.com/docker/docker/api/types/container`  
- `github.com/noxiouz/zapctx/ctxlog` (aliased as `log`)  
- `github.com/sonm-io/core/insonmnia/hardware/gpu` (aliased as `gpu`)  
- `github.com/sonm-io/core/proto` (aliased as `sonm`)  
- `go.uber.org/zap`  
  
External Data/Input Sources:  
- System's DRI devices (collected via `CollectDRICardDevices()`)  
- OpenCL devices (obtained via `gpu.GetGPUDevices()`)  
- Container host configuration (`container.HostConfig`)  
- GPU IDs (`GPUID` slice)  
  
TODOs:  
- None found in the provided code snippet.  
  
### Code Summary  
  
**1. `radeonTuner` Structure:**  
This struct manages Radeon GPU devices. It uses a mutex (`m`) to protect concurrent access to a map (`devMap`) that stores GPU devices identified by their ID (`GPUID`) and their corresponding `sonm.GPUDevice` representation.  
  
**2. `collectDRICardsWithOpenCL()` Function:**  
This function is responsible for collecting DRI (Direct Rendering Infrastructure) cards and matching them with OpenCL devices. It retrieves OpenCL devices using `gpu.GetGPUDevices()`, filters for Radeon devices, collects DRI cards using `CollectDRICardDevices()`, and then attempts to match them based on vendor ID. If a mismatch is found, it returns an error. Finally, it copies the memory size from the OpenCL device to the corresponding DRI device.  
  
**3. `newRadeonTuner()` Function:**  
This function creates a new `radeonTuner` instance. It initializes the device map, calls `collectDRICardsWithOpenCL()` to populate the map with Radeon GPU devices, and logs debug information about each discovered device.  
  
**4. `Tune()` Method:**  
This method tunes a container's host configuration for GPU access. It locks the tuner's mutex, calls `tuneContainer()` (implementation not shown in this snippet) to perform the actual tuning, and then unlocks the mutex.  
  
**5. `Devices()` Method:**  
This method returns a slice of `sonm.GPUDevice` objects stored in the tuner's device map. It locks the mutex before accessing the map and unlocks it afterward.  
  
**6. `Close()` Method:**  
This method is a no-op and returns `nil`. It likely exists to satisfy an interface requirement.  
  
# insonmnia/worker/gpu/remote_server.go  
## GPU Remote Tuner Service Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `context` (standard library)  
*   `github.com/sonm-io/core/proto` (as `sonm`)  
  
**External Data/Input Sources:**  
*   `name` (string): Vendor name passed to `NewRemoteTuner`.  
*   `context.Context`: Used for context propagation.  
*   `sonm.RemoteGPUDeviceRequest`: Input for the `Devices` method.  
  
**TODOs:**  
*   None found in this snippet.  
  
**Code Summary:**  
  
### `NewRemoteTuner` Function  
This function creates a new remote GPU tuner service. It takes a vendor name as input, retrieves the corresponding vendor using `GetVendorByName`, initializes a `Tuner` instance using `New`, and returns a `remoteTunerService` instance. Error handling is included for both vendor retrieval and tuner initialization.  
  
### `remoteTunerService` Struct  
This struct embeds a `Tuner` instance, which likely handles the actual GPU tuning logic.  
  
### `Devices` Method  
This method implements the `Devices` RPC endpoint. It calls the `Devices` method on the embedded `Tuner` instance and returns the result wrapped in a `sonm.RemoteGPUDeviceReply`.  
  
# insonmnia/worker/gpu/remote_tuner.go  
## GPU Remote Tuner Component Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `context`: For managing request contexts.  
*   `fmt`: For formatted printing.  
*   `time`: For time-related operations (e.g., timeouts).  
*   `github.com/docker/docker/api/types/container`: For Docker host configuration.  
*   `github.com/noxiouz/zapctx/ctxlog`: For structured logging with context.  
*   `github.com/sonm-io/core/proto`: Contains protobuf definitions for remote GPU tuner communication.  
*   `github.com/sonm-io/core/util/xgrpc`: For gRPC client creation.  
*   `go.uber.org/zap`: For structured logging.  
  
**External Data/Input Sources:**  
*   `RemoteSocket` (configurable via `Option` interface): The gRPC endpoint for the remote GPU tuner service.  
*   `container.HostConfig`: Docker container host configuration, used for tuning.  
*   `GPUID` slice: List of GPU IDs to tune.  
*   Remote GPU Tuner Service: The external service providing GPU device information and tuning capabilities.  
  
**TODOs:**  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
**1. Remote Tuner Initialization (`newRemoteTuner`)**  
This function establishes a gRPC connection to a remote GPU tuner service using the provided `RemoteSocket`. It retrieves GPU device information from the remote service, logs the detected GPUs (name, memory, device files, driver volumes), and returns a `remoteTuner` instance. Error handling is included for connection failures and remote service access issues.  
  
**2. Device Retrieval (`Devices`)**  
This method retrieves a list of GPU devices from the remote tuner service via gRPC, with a 5-second timeout. It returns a slice of `sonm.GPUDevice` objects.  
  
**3. Device Mapping (`deviceMap`)**  
Creates a map where keys are `GPUID`s and values are corresponding `sonm.GPUDevice` objects. This map is used for efficient lookup of GPU devices by their ID.  
  
**4. GPU Tuning (`Tune`)**  
This function takes a Docker `container.HostConfig` and a slice of `GPUID`s as input. It retrieves the device map using `deviceMap` and then calls the `tuneContainer` function (not shown in this snippet) to perform the actual GPU tuning.  
  
**5. Resource Cleanup (`Close`)**  
This method currently does nothing, but it is included as part of the `Tuner` interface. It could be extended to release resources if needed.  
  
# insonmnia/worker/gpu/tuner_other.go  
## GPU Package - No-GPU Build  
  
**Package Name:** `gpu`  
  
**Imports:**  
  
*   `context`  
  
**External Data/Input Sources:**  
  
*   None. The code relies solely on the build tag `!cl darwin` to determine execution path. No external files, databases, or user input are involved.  
  
**TODOs:**  
  
*   None.  
  
**Code Summary:**  
  
This code snippet provides placeholder implementations for Radeon and Nvidia tuners when the build environment does not support GPU acceleration (specifically, when the `!cl darwin` build tag is active). Both `newRadeonTuner` and `newNvidiaTuner` functions unconditionally return a `NilTuner` instance, effectively disabling GPU-specific tuning logic. This ensures that the package functions correctly even in environments where GPU support is unavailable. The functions take a `context.Context` and optional `Option` arguments, but these are ignored as the return value is always `NilTuner`.  
  
# insonmnia/worker/gpu/utils.go  
## GPU Package Component Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
*   `fmt` (for formatted I/O)  
*   `strings` (for string manipulation)  
*   `github.com/docker/docker/api/types/container` (Docker container configuration types)  
*   `github.com/docker/docker/api/types/mount` (Docker mount types)  
*   `github.com/sonm-io/core/proto` (Sonm core protocol definitions, aliased as `sonm`)  
  
**External Data/Input Sources:**  
*   `container.HostConfig`: Docker host configuration object, modified in-place.  
*   `map[GPUID]*sonm.GPUDevice`: Mapping of GPU IDs to GPU device information from the Sonm core protocol.  
*   `[]GPUID`: Slice of GPU IDs to be bound to the container.  
*   `GPUID`: Custom type (not defined in this snippet, assumed to be a string or similar identifier for GPUs).  
*   `sonm.GPUDevice`: Protocol buffer message representing a GPU device.  
  
**TODOs:**  
*   None found in this snippet.  
  
**Code Summary:**  
  
### Volume Mount Creation  
The `newVolumeMount` function creates a Docker volume mount configuration. It sets the mount type to `TypeVolume`, specifies the source and target paths, sets read-only access, and configures volume options including a driver with a specified name. The driver options are currently empty.  
  
### Container Tuning  
The `tuneContainer` function modifies a Docker `container.HostConfig` to bind GPU devices and volumes to the container. It iterates through a list of GPU IDs, retrieves corresponding device information from a provided map, and adds device mappings to the host configuration. It also parses driver volume pairs (e.g., "host_path:container_path") and creates volume mounts using `newVolumeMount`. Error handling is included for unknown GPU IDs and malformed mount-point strings. The function appends the generated device mappings and volume mounts to the host configuration.  
  
# insonmnia/worker/gpu/volume_plugin.go  
## GPU Volume Plugin Component Summary  
  
**Package Name:** `gpu`  
  
**Imports:**  
- `errors`: For custom error definitions.  
- `fmt`: For formatted string output.  
- `log`: For logging plugin operations.  
- `path`: For path manipulation (joining paths for mountpoints).  
- `regexp`: For volume name validation using regular expressions.  
- `github.com/docker/go-plugins-helpers/volume`: Core volume plugin interface definitions.  
- `github.com/sshaman1101/nvidia-docker/nvidia`: Custom NVIDIA volume management structures and interfaces.  
  
**External Data/Input Sources:**  
- `nvidia.VolumeMap`: An immutable map of NVIDIA volumes, passed during plugin initialization. This map defines the available volumes and their configurations.  
- Volume names provided in requests (Create, List, Get, Remove, Mount, Unmount, Path). These names are expected to follow the format `name_version`.  
  
**TODOs:**  
- None found in the provided code snippet.  
  
---  
  
### Core Functionality: Volume Management  
  
The `gpu` package implements a Docker volume plugin specifically designed for NVIDIA GPU-backed volumes. It provides the necessary methods to create, list, get, remove, mount, unmount, and check capabilities of these volumes. The plugin relies on an external `nvidia.VolumeMap` to manage the actual volume storage and operations.  
  
### Volume Name Parsing and Validation  
  
The `getVolume` function is central to the plugin's operation. It parses volume names using a regular expression (`volVersionRegex`) to extract the volume name and version. It then retrieves the corresponding volume from the `Volumes` map. If the name is invalid or the volume is not found, appropriate errors (`ErrVolumeBadFormat`, `ErrVolumeUnsupported`) are returned.  
  
### Volume Operations  
  
The plugin implements the Docker volume driver interface through the `volumePlugin` struct. The key methods include:  
  
- **Create:** Checks if the volume exists and creates it if it doesn't.  
- **List:** Iterates through the `Volumes` map, listing all available versions for each volume.  
- **Get:** Retrieves volume details (name, mountpoint) if the volume and version exist.  
- **Remove:** Removes a specific version of a volume.  
- **Mount/Path:** Returns the mountpoint for a given volume and version.  
- **Unmount:** Currently just returns the error from `getVolume`.  
- **Capabilities:** Returns the plugin's capabilities (local scope).  
  
### Error Handling  
  
The plugin defines custom errors (`ErrVolumeBadFormat`, `ErrVolumeUnsupported`, `ErrVolumeNotFound`, `ErrVolumeVersion`) to provide specific feedback on volume-related issues. These errors are used throughout the plugin's methods to indicate invalid input or unsupported operations.  
  
