# insonmnia/hardware/hardware.go  
**Package / Component**    
`hardware`  
  
---  
  
### Imports  
```go  
import (  
    "errors"  
    "fmt"  
    "math"  
    "net"  
  
    "github.com/cnf/structhash"  
    "github.com/mohae/deepcopy"  
    "github.com/sonm-io/core/insonmnia/benchmarks"  
    "github.com/sonm-io/core/insonmnia/hardware/cpu"  
    "github.com/sonm-io/core/insonmnia/hardware/ram"  
    "github.com/sonm-io/core/insonmnia/worker/gpu"  
    "github.com/sonm-io/core/proto"  
    "github.com/sonm-io/core/util/netutil"  
)  
```  
  
---  
  
### External Data / Input Sources  
| Source | Description |  
|--------|-------------|  
| `cpu.GetCPUDevice()` | Retrieves CPU device information. |  
| `ram.NewRAMDevice()` | Creates a RAM device instance. |  
| `benchmarks.NetworkIn`, `benchmarks.NetworkOut`, `benchmarks.StorageSize` | Benchmark identifiers used for network and storage metrics. |  
| `structhash.Md5` | Computes MD5 hash of a struct. |  
| `deepcopy.Copy` | Deep copies benchmark structs. |  
| `net.ParseIP`, `netutil.IsPrivateIP`, `netutil.IsIPv4` | IP parsing & classification utilities. |  
  
---  
  
### TODO List  
1. **SetNetworkIncoming** – split NetworkIn into IPv4In and IPv6In.    
2. **AskPlanResources** – make network device use DataSizeRate.    
3. **LimitTo** – refactor the duplicated logic with `ResourcesToBenchmarkMap`.    
  
---  
  
## Summary of Major Code Parts  
  
#### 1. `Hardware` struct  
```go  
type Hardware struct {  
    CPU     *sonm.CPU  
    GPU     []*sonm.GPU  
    RAM     *sonm.RAM  
    Network *sonm.Network  
    Storage *sonm.Storage  
}  
```  
Represents the full hardware profile of a worker’s host, including CPU, GPU array, RAM, network and storage.  
  
#### 2. `NewHardware` – constructor  
* Initializes all sub‑components with empty benchmark maps.  
* Calls external helpers to fill CPU and RAM devices.  
* Returns a ready‑to‑use `*Hardware`.  
  
#### 3. Utility methods  
| Method | Purpose |  
|--------|---------|  
| `LogicalCPUCount()` | Deprecated helper for logical core count. |  
| `Hash()` | Computes a hash of the device mapping via `devicesMap()`. |  
| `HashGPU(indexes []uint64)` | Returns hashes of GPU devices at given indices. |  
| `GPUIDs(gpuResources *sonm.AskPlanGPU)` | Maps GPU benchmark hashes to IDs. |  
  
#### 4. Network configuration  
* **`SetNetworkIncoming(IPs []string)`** – parses IP strings, sets incoming network flags (TODO: IPv4/IPv6 split).    
* **`AskPlanResources()`** – builds an `sonm.AskPlanResources` from current hardware state; includes CPU core percentages, RAM size, storage bytes, GPU hashes and network throughput.    
  
#### 5. Benchmark handling  
* **`SetDevicesFromBenches()`** – pulls benchmark results for network in/out and storage into the struct.  
* **`insertBenches(to map[uint64]*sonm.Benchmark, from map[uint64]*sonm.Benchmark, proportion float64)`** – helper that merges a set of benchmarks with a given proportion.  
* **`insertBench(...)`** – low‑level merge logic for a single benchmark.  
  
#### 6. Full benchmark conversion  
* **`FullBenchmarks()`** – converts current resources into a `sonm.Benchmarks` slice.  
* **`ResourcesToBenchmarks(resources *sonm.AskPlanResources)`** – orchestrates the conversion by first building a map and then flattening it.  
  
#### 7. Mapping helpers  
* **`LimitTo(resources *sonm.AskPlanResources)`** – similar to `ResourcesToBenchmarkMap`, but stores benchmarks directly into device structs; TODO: refactor.  
* **`ResourcesToBenchmarkMap(resources *sonm.AskPlanResources)`** – builds a map of benchmark results from the given resources.  
  
#### 8. Device mapping struct  
```go  
type hashableRAM struct {  
    Available uint64 `json:"available"`  
}  
  
type DeviceMapping struct {  
    CPU        *sonm.CPUDevice     `json:"cpu"`  
    GPU        []*sonm.GPUDevice   `json:"gpu"`  
    RAM        hashableRAM         `json:"ram"`  
    NetworkIn  uint64              `json:"network_in"`  
    NetworkOut uint64              `json:"network_out"`  
    Storage    *sonm.StorageDevice `json:"storage"`  
    NetFlags   *sonm.NetFlags      `json:"netflags"`  
}  
```  
Provides a lightweight representation of the hardware for hashing and serialization.  
  
#### 9. Hashing & mapping  
* **`(dm *DeviceMapping) Hash()`** – MD5 hash string of the device mapping.  
* **`(h *Hardware) devicesMap()`** – builds a `DeviceMapping` from the current `Hardware`, used by `Hash()`.  
  
---  
  
All functions are grouped logically: initialization, network handling, benchmark conversion, and mapping. The code is ready for further refactoring (TODOs noted).  
  
# insonmnia/hardware/hardware_test.go  
## Package / Component    
**hardware**  
  
### Imports  
```go  
import (  
	"testing"  
  
	"github.com/sonm-io/core/insonmnia/benchmarks"  
	"github.com/sonm-io/core/proto"  
	"github.com/stretchr/testify/assert"  
	"github.com/stretchr/testify/require"  
)  
```  
  
* `testing` – standard Go testing package.    
* `github.com/sonm-io/core/insonmnia/benchmarks` – contains benchmark identifiers and helper functions.    
* `github.com/sonm-io/core/proto` – provides the core data structures (`Hardware`, `CPUDevice`, etc.).    
* `github.com/stretchr/testify/assert` & `require` – assertion helpers for unit tests.  
  
### External Data / Input Sources  
| Source | Description |  
|--------|-------------|  
| `NewHardware()` | Factory that creates a new `Hardware` instance. |  
| `sonm.CPUDevice`, `sonm.StorageDevice`, `sonm.GPUDevice`, `sonm.Benchmark` | Types used to populate the hardware structure. |  
| `benchmarks.RamSize` | Benchmark key for RAM size. |  
  
### TODOs  
* **Dev-770** – comment in `TestHardware_ResourcesToBenchmarks`.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. `getTestHardware`  
Creates a fully‑populated test instance of `Hardware`.    
* Sets RAM device availability to 1024, CPU model “Intel” with 2 cores and 1 socket, network in/out bandwidths, storage bytes, and one GPU entry.    
* Returns the constructed object for use by other tests.  
  
### 2. `TestHardwareHash`  
Validates that hashing a hardware instance is deterministic.    
* Calls `getTestHardware`, hashes it once (`hash1`), adds several benchmark entries (CPU, Network, Storage, RAM, GPU) and hashes again (`hash2`).    
* Asserts that both hashes are non‑empty and equal.  
  
### 3. `TestHardwareLimitTo`  
Tests the `LimitTo` method of a hardware instance.    
* Adds four CPU benchmarks with different splitting algorithms (MAX, MIN, NONE, PROPORTIONAL).    
* Creates an `AskPlanResources` object specifying 150 % core usage.    
* Calls `hardware.LimitTo(resources)` and verifies that each benchmark result matches the expected values (the last one should be 75 due to proportional split).  
  
### 4. `TestHardware_ResourcesToBenchmarks`  
Dev‑770 test for converting resource plans into benchmarks.    
* Modifies RAM availability, sets a RAM benchmark using the `RamSize` key, and defines an `AskPlanRAM` with a data size of 4194304 bytes.    
* Calls `hardware.ResourcesToBenchmarks(resources)` and checks that the resulting benchmark map contains the expected value for the RAM size key.  
  
---  
  
All tests rely on the `sonm` package types and ensure that hashing, limiting, and resource‑to‑benchmark conversion work as intended.  
  
# insonmnia/hardware/marshal.go  
# Package / Component    
**Name:** `hardware`    
  
## Imports  
```go  
import (  
	"github.com/sonm-io/core/proto"  
)  
```  
The file pulls in the *proto* package from the sonm‑IO core library, which provides the `sonm.DevicesReply` type used by the method below.  
  
---  
  
## External Data / Input Sources    
* The method consumes a receiver of type `*Hardware`.    
* It produces a pointer to a `sonm.DevicesReply`, mapping fields from the Hardware struct into the reply structure.    
  
---  
  
## TODOs  
No explicit TODO comments are present in this file.  
  
---  
  
# Summary of Major Code Parts  
  
### 1. `IntoProto` Method    
- **Signature**: `func (h *Hardware) IntoProto() *sonm.DevicesReply`  
- **Purpose**: Convert the internal `Hardware` representation into a protocol‑buffer reply that can be sent over the network or persisted.  
- **Implementation Details**:  
  - Returns a pointer to a new `sonm.DevicesReply`.  
  - Populates each field (`CPU`, `GPUs`, `RAM`, `Network`, `Storage`) directly from the corresponding fields of the receiver `h`.    
  - The mapping is straightforward and assumes that the struct tags in `Hardware` match those expected by `sonm.DevicesReply`.  
  
---  
  
