## Package/Component Name: lsgpu

**Project Package Structure:**

```
cmd/lsgpu/
├── main.go
```

**Code Summary:**

The `lsgpu` package is a command-line utility designed to list and display information about GPU devices present in the system. It leverages the `github.com/sonm-io/core/insonmnia/worker/gpu` package to collect device details, including temperature, fan speed, power consumption, vendor ID, device ID, major/minor versions, PCI bus ID, and associated devices.

The program begins by printing its version string (`appVersion`). It then calls `gpu.CollectDRICardDevices()` to retrieve a list of GPU cards. If this call fails, the program exits with an error message. For each detected card, it prints detailed information including metrics (temperature, fan speed, power) if available and device identifiers (VID, DID, major/minor versions, PCI bus ID). Finally, it iterates through associated devices for each card printing their names.

The output is formatted to be human-readable, providing a clear overview of the GPU hardware present in the system. The code handles potential errors during metric retrieval gracefully by printing "metrics is not available" instead of crashing.

**Configuration:**

*   `appVersion`: Externally defined version string (likely through build flags or environment variables).
*   No other configuration options are explicitly exposed via command-line arguments, files, or environment variables. The application relies solely on the system's GPU hardware for input.

**Edge Cases/Launch Conditions:**

The application is a standard executable; it can be launched directly from the command line without any special flags or arguments. If `gpu.CollectDRICardDevices()` fails (e.g., due to missing drivers, insufficient permissions, or no GPUs present), the program will exit with an error message. The behavior in such cases is well-defined: the application terminates gracefully after printing the error.