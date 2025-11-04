# insonmnia/benchmarks/benchmarks.go  
## Benchmarks Package Component Summary  
  
**Package Name:** `benchmarks`  
  
**Imports:**  
- `context`  
- `encoding/json`  
- `fmt`  
- `io`  
- `net/http`  
- `net/url`  
- `os`  
- `github.com/noxiouz/zapctx/ctxlog`  
- `github.com/sonm-io/core/proto` (aliased as `sonm`)  
- `go.uber.org/zap`  
  
**External Data/Input Sources:**  
- **URL/File Path:** The package loads benchmark data from either an HTTP/HTTPS URL or a local file. The URL/path is configurable via the `Config` struct.  
- **JSON Data:** The benchmark data is expected to be in JSON format, containing a map of benchmark definitions.  
- **Environment Variables:** The package uses environment variables `SONM_BENCHMARK_ID`, `SONM_CPU_COUNT`, and `SONM_GPU_TYPE` for configuration.  
  
**TODOs:**  
- No TODO comments found in the provided code.  
  
**Code Summary:**  
  
### Benchmark List Loading and Parsing  
The core functionality revolves around loading and parsing benchmark data from external sources (URLs or files). The `benchmarkList` struct stores the benchmarks in multiple maps (by ID, code, and device type) for efficient access. The `load` function handles fetching the data, while `readResults` parses the JSON and populates the internal maps. Error handling is present for URL parsing, HTTP requests, file opening, and JSON decoding.  
  
### Benchmark List Interface  
The `BenchList` interface defines methods for accessing the loaded benchmarks: `Max` (returns the maximum benchmark ID), `ByID` (returns a slice of benchmarks by ID), `MapByDeviceType` (returns a map of benchmarks grouped by device type), and `MapByCode` (returns a map of benchmarks grouped by code).  
  
### Configuration and Initialization  
The `NewBenchmarksList` function initializes the benchmark list by loading data from a configured URL (provided in the `Config` struct). If the URL is empty, the benchmark list is initialized as empty.  
  
### Data Structures  
- `ResultJSON`: Represents the result of a single benchmark execution, containing the result value and the device ID.  
- `ContainerBenchmarkResultsJSON`: Represents the JSON structure expected from a container executing benchmarks, mapping benchmark codes to result structs.  
- `Config`: Holds the configuration for loading the benchmark list, specifically the URL.  
  
### Utility Functions  
- `loadURL`: Fetches benchmark data from a URL using HTTP GET.  
- `loadFile`: Loads benchmark data from a local file.  
  
# insonmnia/benchmarks/mapping.go  
## Benchmarks Component - Loader File Summary  
  
**Package Name:** `benchmarks`  
  
**Imports:**  
*   `context` (standard library)  
*   `github.com/sonm-io/core/proto` (sonm-io/core)  
  
**External Data/Input Sources:**  
*   Configuration URI (string) passed to `NewLoader` to initialize the `Config` struct.  
*   Benchmarks data loaded from the URI via `NewBenchmarksList`.  
  
**TODOs:**  
*   None found in this file.  
  
**Code Summary:**  
  
### Mapping Interface and Implementations  
This section defines the `Mapping` interface, which provides methods to retrieve `DeviceType` and `SplittingAlgorithm` based on an ID. Two implementations are provided: `mapping` (using maps) and `arrayMapping` (using slices). The `mapping` implementation uses maps for efficient lookup, while `arrayMapping` uses slices for potentially better performance when the ID range is dense and small.  
  
### Loader Interface and Implementation  
The `Loader` interface defines a `Load` method that retrieves a `Mapping`. The `loader` struct implements this interface, taking a configuration URI. The `Load` method creates a `BenchmarksList` from the URI, determines whether to use an array or map-based mapping based on the maximum benchmark ID, and returns the appropriate mapping implementation.  
  
### Mapping Creation Functions  
`NewArrayMapping` creates an `arrayMapping` by populating slices with device types and splitting algorithms from the `BenchmarksList`. `NewMapMapping` creates a `mapping` by populating maps with the same data.  
  
### Data Structures  
The code defines `Config`, `Loader`, `Mapping`, `mapping`, `arrayMapping`, `BenchList` (assumed to be defined elsewhere) structs to manage benchmark data and provide lookup functionality. The `deviceTypes` and `splittingAlgorithms` fields are central to the mapping implementations.  
  
The code handles out-of-bounds ID lookups by returning default values (`sonm.DeviceType_DEV_UNKNOWN` and `sonm.SplittingAlgorithm_NONE`). The `arrayMappingThreshold` constant determines when to switch from array-based to map-based mapping.  
  
