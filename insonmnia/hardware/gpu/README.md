# gpu package

## Project structure
```
insonmnia/hardware/gpu/
├── cl.go          (OpenCL implementation – compiled when tag `cl` is set)
├── cl_other.go    (fallback implementation – compiled when tag `!cl` is set)
└── device.go      (public API wrapper around the backend call)
```

## Build tags, flags and command‑line arguments
| Item | Value |
|------|-------|
| **Build tags** | `cl` for the OpenCL implementation; `!cl` for the fallback. |
| **cgo directives** |  
```go
// #cgo darwin LDFLAGS: -framework OpenCL
// #cgo linux  LDFLAGS:-lOpenCL
```
Link against the macOS framework or Linux library automatically. |
| **Environment variables / constants** | `maxPlatforms = 32`, `maxDeviceCount = 64`, `CL_PLATFORM_NOT_FOUND_KHR = C.cl_int(-1001)` – used for array sizes and error handling. |
| **Command‑line flags** | Use `-tags=cl` to build the OpenCL version, or omit it (or use `-tags=!cl`) to build the fallback. |

## Summary of package logic
* **`device.go`** exposes a single public function  
  ```go
  func GetGPUDevices() ([]*sonm.GPUDevice, error)
  ```  
  which simply forwards its result from `GetGPUDevicesUsingOpenCL`. It is the entry point for other parts of the project.

* **`cl.go`** implements `GetGPUDevicesUsingOpenCL`.  
  * Enumerates all OpenCL platforms (`getPlatforms()`), then for each platform calls `platform.getGPUDevices()` to obtain a slice of `clDevice`.  
  * For every device it pulls name, vendor name, vendor ID and global memory size via helper methods (`deviceName()`, `vendorName()`, etc.) and appends a new `sonm.GPUDevice` (from the local `proto` package) into the result slice.  
  * Handles errors with `errorToString(err C.cl_int)`.

* **`cl_other.go`** is compiled when the build tag `cl` is not set; it contains a stub for `GetGPUDevicesUsingOpenCL`. It can be replaced later by an alternative backend (e.g., Vulkan, DirectX) or a pure‑Go implementation.

## Edge cases / launch scenarios
| Scenario | How to invoke |
|----------|---------------|
| **Native OpenCL** | Build with `-tags=cl` (`go build -tags=cl ./...`). The resulting binary will use the OpenCL backend. |
| **Fallback** | Build without the tag or with `-tags=!cl`. The binary will use the stub implementation from `cl_other.go`. |
| **Cross‑platform** | On macOS, the linker flag `-framework OpenCL` is used; on Linux it links against `libOpenCL.so`. No extra environment variables are required beyond those constants. |

The package therefore provides a thin wrapper around the OpenCL API that enumerates GPU devices and returns them as Go structs.