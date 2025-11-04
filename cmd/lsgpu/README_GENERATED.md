# Package: `lsgpu`

## Project Package Structure:

```
cmd/lsgpu/
├── main.go
```

## Summary:

The `lsgpu` package is a command-line utility designed to list and display information about GPU devices present in the system. It leverages the `github.com/sonm-io/core/insonmnia/worker/gpu` package to enumerate and retrieve metrics from these devices.

## Configuration:

*   **Environment Variables:** None explicitly used in the provided code.
*   **Flags/Cmdline Arguments:** None. The application runs without any command-line arguments.
*   **Files/Paths:** The application interacts with system GPU devices, but no specific file paths are used for configuration.

## Launch Edgecases:

The application is launched directly via `go run main.go`. No special launch conditions are apparent.

## Code Logic:

1.  The program prints its version (`appVersion`).
2.  It calls `gpu.CollectDRICardDevices()` to obtain a list of GPU devices.
3.  If device collection fails, the program exits with an error.
4.  For each detected GPU device, it prints:
    *   Path
    *   Temperature
    *   Fan speed
    *   Power consumption (if available)
    *   Vendor/Device IDs
    *   Major/Minor versions
    *   PCI bus ID
    *   Related devices
5.  If metrics retrieval fails for a device, it prints "metrics is not available".

## Relations:

The `main` package depends on the `github.com/sonm-io/core/insonmnia/worker/gpu` package for GPU device interaction. The `gpu.CollectDRICardDevices()` function is central to the application's functionality.