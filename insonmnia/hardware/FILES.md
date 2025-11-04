# insonmnia/hardware/hardware.go  
## Hardware Package Component Summary  
  
**Package Name:** `hardware`  
  
**Imports:**  
  
*   `errors`  
*   `fmt`  
*   `math`  
*   `net`  
*   `github.com/cnf/structhash`  
*   `github.com/mohae/deepcopy`  
*   `github.com/sonm-io/core/insonmnia/benchmarks`  
*   `github.com/sonm-io/core/insonmnia/hardware/cpu`  
*   `github.com/sonm-io/core/insonmnia/hardware/ram`  
*   `github.com/sonm-io/core/insonmnia/worker/gpu`  
*   `github.com/sonm-io/core/proto` (as `sonm`)  
*   `github.com/sonm-io/core/util/netutil`  
  
**External Data/Input Sources:**  
  
*   CPU device information obtained from `cpu.GetCPUDevice()`.  
*   RAM device information obtained from `ram.NewRAMDevice()`.  
*   Network IP addresses provided as strings to `SetNetworkIncoming()`.  
*   GPU resources (hashes) provided via `sonm.AskPlanGPU` to `GPUIDs()`.  
*   AskPlanResources provided to `LimitTo()` and `ResourcesToBenchmarkMap()`.  
  
**TODOs:**  
  
*   `// TODO: split NetworkIn into IPv4In and IPv6In.` in `SetNetworkIncoming()`.  
*   `//TODO: Make network device use DataSizeRate` in `AskPlanResources()`.  
*   `// TODO: find a way to refactor all this shit.` in `LimitTo()`.  
  
**Code Summaries:**  
  
### Hardware Struct  
  
The `Hardware` struct aggregates information about the system's CPU, GPU, RAM, network, and storage. It uses nested `sonm` types to represent these components.  
  
### NewHardware Function  
  
The `NewHardware()` function initializes a `Hardware` instance, retrieving CPU and RAM device information using external functions (`cpu.GetCPUDevice()`, `ram.NewRAMDevice()`). It pre-allocates maps for benchmarks within CPU, RAM, Network, and Storage.  
  
### LogicalCPUCount Function  
  
The `LogicalCPUCount()` function (deprecated) returns the number of logical CPUs.  
  
### Hash Functions  
  
The `Hash()` and `HashGPU()` functions generate hash strings representing the hardware configuration. `Hash()` uses a `DeviceMapping` to create a hash of CPU, GPU, RAM, and network information. `HashGPU()` generates hashes for specific GPU indexes.  
  
### GPU ID Retrieval  
  
The `GPUIDs()` function maps GPU resources (hashes) to GPU IDs, returning a slice of `gpu.GPUID` values.  
  
### Network Configuration  
  
The `SetNetworkIncoming()` function checks if incoming IP addresses are private IPv4 addresses and sets the `NetFlags` accordingly.  
  
### Resource Reporting  
  
The `AskPlanResources()` function creates a `sonm.AskPlanResources` instance, populating it with CPU core counts, RAM size, storage size, GPU hashes, and network flags.  
  
### Benchmark Handling  
  
The `SetDevicesFromBenches()` function populates network and storage benchmarks from existing benchmark data. The `insertBenches()` and `insertBench()` functions handle merging benchmarks with different splitting algorithms (NONE, PROPORTIONAL, MAX, MIN).  
  
### Benchmark Conversion  
  
The `FullBenchmarks()` and `ResourcesToBenchmarks()` functions convert hardware resources to benchmark slices.  
  
### Resource Limiting  
  
The `LimitTo()` function creates a new `Hardware` instance with limited resources based on provided `AskPlanResources`. It proportionally scales benchmarks for CPU, storage, RAM, and network.  
  
### Resource to Benchmark Map  
  
The `ResourcesToBenchmarkMap()` function converts hardware resources to a benchmark map.  
  
### Device Mapping  
  
The `DeviceMapping` struct and its `Hash()` method provide a hashable representation of hardware devices.  
  
# insonmnia/hardware/hardware_test.go  
## Hardware Package Component Summary  
  
**Package Name:** `hardware`  
  
**Imports:**  
  
*   `testing`  
*   `github.com/sonm-io/core/insonmnia/benchmarks`  
*   `github.com/sonm-io/core/proto` (aliased as `sonm`)  
*   `github.com/stretchr/testify/assert`  
*   `github.com/stretchr/testify/require`  
  
**External Data/Input Sources:**  
  
*   `sonm.AskPlanResources`: Used for limiting hardware resources. Contains CPU, RAM, and other resource specifications.  
*   `sonm.Benchmark`: Represents benchmark results for different hardware components.  
*   `sonm.CPUDevice`, `sonm.StorageDevice`, `sonm.GPUDevice`: Structures defining hardware device properties.  
*   `sonm.DataSize`: Represents size in bytes.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
---  
  
### Hardware Creation and Initialization  
  
The `getTestHardware` function creates a `Hardware` instance with predefined values for RAM, CPU, Network, Storage, and GPU. It sets available resources and device properties for testing purposes.  
  
### Hardware Hashing  
  
The `TestHardwareHash` function verifies that the `Hash` method returns a non-empty hash value. It then adds benchmark results to various hardware components (CPU, Network, Storage, RAM, GPU) and asserts that the hash remains consistent after the benchmarks are added. This suggests the hash is based on hardware configuration *and* benchmark data.  
  
### Resource Limiting  
  
The `TestHardwareLimitTo` function tests the `LimitTo` method, which restricts hardware resources based on an input `sonm.AskPlanResources`. It sets up CPU benchmarks with different splitting algorithms and then limits the CPU usage to 150% of the total cores. The assertion verifies that the benchmark results are adjusted accordingly.  
  
### Resources to Benchmarks Conversion  
  
The `TestHardware_ResourcesToBenchmarks` function tests the `ResourcesToBenchmarks` method, which converts resource requests (e.g., RAM size) into benchmark values. It sets the available RAM and then requests a specific amount of RAM. The assertion confirms that the resulting benchmark value matches the requested RAM size.  
  
# insonmnia/hardware/marshal.go  
## Hardware Package Component Summary  
  
**Package Name:** `hardware`  
  
**Imports:**  
  
*   `github.com/sonm-io/core/proto` (specifically `sonm` package)  
  
**External Data/Input Sources:**  
  
*   The component relies on the `Hardware` struct (not shown in this snippet, but assumed to exist within the package) which contains fields for CPU, GPU, RAM, Network, and Storage. These fields are the primary input data.  
  
**TODOs:**  
  
*   None found in this snippet.  
  
**Code Summary:**  
  
### `IntoProto()` Function  
  
This function converts the internal `Hardware` struct's data into a `sonm.DevicesReply` protobuf message. It maps the `CPU`, `GPU`, `RAM`, `Network`, and `Storage` fields of the `Hardware` struct to the corresponding fields in the protobuf message. This function is likely used for serializing hardware information for communication with other parts of the system (e.g., for RPC calls or data storage).  
  
