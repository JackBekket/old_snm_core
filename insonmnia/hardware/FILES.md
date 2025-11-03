# insonmnia/hardware/hardware.go  
## Hardware Package Summary  
  
**Package Name:** `hardware`  
  
**Imports:**  
  
*   `errors`: For error handling.  
*   `fmt`: For formatted I/O.  
*   `math`: For mathematical functions (Ceil, NaN, Inf).  
*   `net`: For network-related operations (IP parsing).  
*   `github.com/cnf/structhash`: For hashing structs.  
*   `github.com/mohae/deepcopy`: For deep copying data structures.  
*   `github.com/sonm-io/core/insonmnia/benchmarks`: Custom benchmark definitions.  
*   `github.com/sonm-io/core/insonmnia/hardware/cpu`: CPU hardware details.  
*   `github.com/sonm-io/core/insonmnia/hardware/ram`: RAM hardware details.  
*   `github.com/sonm-io/core/insonmnia/worker/gpu`: GPU worker definitions.  
*   `github.com/sonm-io/core/proto`: Protocol buffer definitions (e.g., `sonm`).  
*   `github.com/sonm-io/core/util/netutil`: Network utility functions (IP validation).  
  
**External Data Sources:**  
  
*   CPU device information obtained from `cpu.GetCPUDevice()`.  
*   RAM device information obtained from `ram.NewRAMDevice()`.  
*   Network IP addresses for private IP detection (`SetNetworkIncoming`).  
*   GPU hashes and IDs used in resource allocation (`HashGPU`, `GPUIDs`).  
  
**TODOs:**  
  
*   Split `NetworkIn` into IPv4/IPv6 specific fields.  
*   Refactor the repeated benchmark-related logic to avoid code duplication (mentioned in `LimitTo` and `ResourcesToBenchmarkMap`).  
*   Update network device usage to use `DataSizeRate`.  
  
### Core Structures  
  
**Hardware:** Represents accumulated hardware information, including CPU, GPU, RAM, Network, and Storage.  Uses nested `sonm.*` structs for detailed specifications.  
  
**DeviceMapping:** A simplified structure used for hashing hardware configurations (CPU, GPU, RAM, network flags). Excludes dynamic values like throughput to ensure consistent hashes.  
  
### Key Functions  
  
**NewHardware():** Initializes a `Hardware` struct with default values and retrieves CPU/RAM device information using external functions (`cpu.GetCPUDevice`, `ram.NewRAMDevice`).  
  
**LogicalCPUCount():** (Deprecated) Returns the number of logical CPUs in the system.  
  
**Hash():** Generates an MD5 hash of the hardware configuration using `structhash`.  
  
**HashGPU():** Calculates hashes for specified GPU indexes, returning a slice of strings or an error if an index is invalid.  
  
**GPUIDs():** Maps GPU resources (hashes) to their corresponding IDs within the system. Returns an error if a resource hash cannot be found.  
  
**SetNetworkIncoming():** Flags network as incoming if private IPv4 addresses are detected in provided IPs.  
  
**AskPlanResources():** Creates `sonm.AskPlanResources` based on hardware capabilities, including CPU cores, RAM size, storage availability, GPU hashes, and network throughput.  
  
**LimitTo():** Restricts the Hardware struct to match passed resources (CPU, Storage, RAM, Network). It applies proportional scaling of benchmarks if necessary. Returns an error if resource constraints are violated or unknown GPU hashes are provided.  
  
**ResourcesToBenchmarkMap():** Converts `sonm.AskPlanResources` into a map of benchmark IDs and results. Handles normalization checks for GPU resources before processing.  
  
**FullBenchmarks() & ResourcesToBenchmarks():** Convert hardware resources to benchmarks using the internal logic, returning them in different formats (slice vs map).  
  
### Benchmark Handling  
  
The package heavily relies on `sonm.Benchmark` structs to represent performance metrics. Functions like `insertBenches`, `insertBench`, and various resource-to-benchmark conversion methods handle benchmark aggregation based on splitting algorithms (`NONE`, `PROPORTIONAL`, `MAX`, `MIN`). These functions ensure benchmarks are correctly scaled or merged when limiting hardware resources.  
  
# insonmnia/hardware/hardware_test.go  
## Hardware Package Summary  
  
**Package Name:** `hardware`  
  
**Imports:**  
  
*   `github.com/sonm-io/core/insonmnia/benchmarks`: Used for benchmark IDs (e.g., `benchmarks.RamSize`).  
*   `github.com/sonm-io/core/proto`: Contains definitions for core data structures like `Hardware`, `CPUDevice`, `StorageDevice`, `Benchmark`, and related types used throughout the package.  
*   `github.com/stretchr/testify/assert`: Used for assertions in tests.  
*   `github.com/stretchr/testify/require`: Used for requiring conditions to be true in tests, panicking if not.  
*   `testing`: Standard Go testing library.  
  
**External Data / Inputs:**  
  
The package relies on `sonm.AskPlanResources` structures (CPU, RAM) as input for resource limiting and benchmark conversion functions.  It also uses hardcoded values within test cases to simulate hardware configurations (RAM size, CPU cores, storage capacity). The tests depend heavily on the correctness of the underlying `proto` definitions.  
  
**TODOs:**  
  
No explicit TODO comments are present in this code snippet.  
  
---  
  
### Core Functionality Breakdown:  
  
#### Hardware Creation & Initialization (`getTestHardware`)  
  
This function creates a new `Hardware` instance and populates it with test values for RAM, CPU, Network, Storage, and GPU resources.  It's used as a setup routine for most tests in the package. The created hardware has predefined available resources (RAM: 1024, CPU: Intel with 2 cores, Network In/Out: 100/200, Storage: 100500 bytes).  
  
#### Hardware Hashing (`TestHardwareHash`)  
  
The `Hash()` method is tested to ensure it generates a non-empty hash value for the hardware configuration. The test verifies that modifying benchmarks on different device types (CPU, Network, Storage, RAM, GPU) and then re-hashing produces the same result as the initial hash, indicating consistency in hashing logic.  
  
#### Resource Limiting (`TestHardwareLimitTo`)  
  
The `LimitTo()` method is tested with a CPU resource request of 150% core usage. The test verifies that benchmarks on the CPU are correctly adjusted based on this limit (e.g., if total cores are 2, then 150% would result in a reduced benchmark value).  It checks for correct handling of different splitting algorithms (`MAX`, `MIN`, `NONE`, `PROPORTIONAL`).  
  
#### Resource Conversion to Benchmarks (`TestHardware_ResourcesToBenchmarks`)  
  
This test verifies that the `ResourcesToBenchmarks()` method correctly converts an `AskPlanResources` structure (specifically RAM size) into a corresponding benchmark entry within the hardware's configuration. It checks if the resulting benchmark value matches the requested resource amount. The function uses predefined benchmark ID (`benchmarks.RamSize`).  
  
---  
  
This summary provides a high-level overview of the key functions and tests within the `hardware` package, focusing on its core functionality related to hardware representation, hashing, resource limiting, and conversion between resources and benchmarks.  The code relies heavily on external definitions from the `sonm/proto` package for data structures and types.  
  
# insonmnia/hardware/marshal.go  
## Hardware Package Summary  
  
**Package Name:** `hardware`  
  
**Imports:**  
  
*   `github.com/sonm-io/core/proto`: Used for defining the data structures exchanged with other components (specifically, the `DevicesReply` type).  
  
**External Data / Input Sources:**  
  
The package relies on an instance of a `Hardware` struct which contains fields representing CPU, GPU, RAM, Network and Storage. The values within this struct are assumed to be populated externally before calling the `IntoProto()` method. No direct external input is handled by the code itself; it transforms internal data into a protobuf representation.  
  
**TODOs:** None present in provided snippet.  
  
### Code Summary: `IntoProto()` Method  
  
The primary function, `IntoProto()`, converts an instance of the `Hardware` struct into a `sonm.DevicesReply` protobuf message. It maps the fields (CPU, GPUs, RAM, Network, Storage) from the `Hardware` struct to corresponding fields in the `sonm.DevicesReply` structure. This method serves as an interface for serializing hardware configuration data into a format suitable for communication with other parts of the system.  
  
