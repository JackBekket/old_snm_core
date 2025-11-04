## GPU Package Summary

This package aims to enumerate GPU devices using OpenCL. The core logic resides in `cl.go`, which initializes OpenCL bindings and retrieves GPU devices via `clGetDeviceIDs`. Device information (name, vendor, memory) is extracted using `clGetDeviceInfo` and packaged into `sonm.GPUDevice` structs. Error handling is robust, mapping OpenCL error codes to human-readable strings.

`cl_other.go` contains a stub implementation of `GetGPUDevicesUsingOpenCL` that always returns an error (`ErrUnsupportedPlatform`) when compiled without OpenCL support (indicated by the `// +build !cl` directive). This suggests conditional compilation based on OpenCL availability.

`device.go` defines a custom error `ErrUnsupportedPlatform` and provides a wrapper function `GetGPUDevices` that calls `GetGPUDevicesUsingOpenCL`.

**Configuration:**

*   **Build Flags:** The `// +build !cl` directive in `cl_other.go` controls whether OpenCL support is included during compilation.
*   **Environment Variables:** None explicitly used, but OpenCL drivers and libraries must be installed on the system.
*   **Cmdline Arguments:** None.

**Edge Cases:**

*   If OpenCL is not installed or configured correctly, `GetGPUDevicesUsingOpenCL` in `cl.go` may fail, leading to errors.
*   If compiled without OpenCL support (`// +build !cl`), `GetGPUDevices` will always return `ErrUnsupportedPlatform`.
*   The code assumes OpenCL is available and properly configured. No fallback mechanisms are implemented if OpenCL fails.

**Project Structure:**

```
insonmnia/
└── hardware/
    └── gpu/
        ├── cl.go
        ├── cl_other.go
        └── device.go
```

**Relations:**

*   `device.go` depends on `cl.go` for actual GPU device enumeration.
*   `cl_other.go` provides a conditional stub for OpenCL support.
*   All files rely on the `sonm.GPUDevice` struct from `github.com/sonm-io/core/proto`.