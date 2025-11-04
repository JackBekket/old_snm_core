## `benchmarks` Package Summary

This package provides functionality for loading, parsing, and accessing benchmark data used for performance evaluation, likely within a distributed computing or resource management system (based on the `sonm-io/core/proto` import). The core logic revolves around fetching benchmark definitions from either a URL or a local file in JSON format. These definitions are then stored in efficient data structures (maps and slices) to allow quick retrieval based on benchmark ID, code, or device type. The package also includes interfaces and implementations for mapping benchmark IDs to device types and splitting algorithms, optimizing lookup performance based on the range of IDs.

**Configuration:**

*   **Environment Variables:**
    *   `SONM_BENCHMARK_ID`:  Potentially used for filtering or selecting specific benchmarks.
    *   `SONM_CPU_COUNT`:  Likely used to adjust benchmark parameters based on CPU availability.
    *   `SONM_GPU_TYPE`:  Used to select benchmarks appropriate for the available GPU.
*   **Cmdline Arguments/Flags:** None explicitly defined in the provided code. Configuration is primarily driven by the `Config` struct.
*   **Files:**
    *   Benchmark data file (JSON format): Path specified in the `Config` struct's URL field.
*   **URLs:**
    *   Benchmark data URL: Configured via the `Config` struct.

**Edge Cases (Launch/Execution):**

*   If the `Config` URL is empty, the benchmark list is initialized as empty. This could lead to errors if benchmarks are expected but not loaded.
*   The package relies on external JSON data. Invalid JSON format or missing fields will cause parsing errors.
*   The `arrayMappingThreshold` constant determines when to switch between array-based and map-based mapping. Incorrect tuning of this threshold could degrade performance.

**Project Package Structure:**

```
insonmnia/
├── benchmarks/
│   ├── benchmarks.go
│   └── mapping.go
```

**Relationships:**

*   `benchmarks.go` handles the core loading and parsing of benchmark data.
*   `mapping.go` provides interfaces and implementations for mapping benchmark IDs to device types and splitting algorithms, using data loaded by `benchmarks.go`.
*   The `Config` struct in both files centralizes configuration.
*   The `sonm.DeviceType` and `sonm.SplittingAlgorithm` types from `github.com/sonm-io/core/proto` are used throughout the package, indicating integration with a larger system.