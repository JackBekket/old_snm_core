# insonmnia/hardware/gpu/cl.go  
# Package / Component    
**Name:** `gpu`    
  
## Imports  
```go  
import "C"  
  
import (  
    "fmt"  
    "strconv"  
    "unsafe"  
  
    "github.com/sonm-io/core/proto"  
)  
```  
  
* `C` – cgo interface for OpenCL functions.    
* Standard library packages: `fmt`, `strconv`, `unsafe`.    
* Local package: `github.com/sonm-io/core/proto` (provides the `GCPUDevice` struct used in this file).  
  
---  
  
## External data / input sources  
| Source | Description |  
|--------|-------------|  
| OpenCL API | The code calls `clGetPlatformIDs`, `clGetDeviceIDs`, and various `clGetDeviceInfo` functions to enumerate platforms, devices, and device properties. |  
| Constants | `maxPlatforms = 32`, `maxDeviceCount = 64`, `CL_PLATFORM_NOT_FOUND_KHR = C.cl_int(-1001)` – used for array sizes and error handling. |  
  
---  
  
## TODOs  
No explicit `TODO` comments were found in the provided snippet.  
  
---  
  
## Summary of major code parts  
  
### Build tags & cgo directives    
```go  
// +build cl  
  
package gpu  
  
// #cgo darwin LDFLAGS: -framework OpenCL  
// #cgo linux LDFLAGS:-lOpenCL  
...  
```  
*Build tag `cl` ensures this file is compiled only when the build flag `cl` is set.*    
The cgo directives link against the OpenCL framework on macOS and the OpenCL library on Linux. Conditional includes load the correct header files for each platform.  
  
### Platform abstraction (`platform`)    
```go  
type platform struct {  
    id C.cl_platform_id  
}  
```  
* Holds a platform identifier returned by `clGetPlatformIDs`.*    
  
#### `getPlatforms()`    
* Calls `C.clGetPlatformIDs` to fetch up to 32 platforms, stores them in a slice of pointers to `platform`.    
* Handles errors via the helper `errorToString`.    
  
### Device abstraction (`clDevice`)    
```go  
type clDevice struct {  
    id C.cl_device_id  
}  
```  
* Holds a device identifier returned by `C.clGetDeviceIDs`.*    
  
#### Methods for retrieving device properties    
| Method | Purpose |  
|--------|---------|  
| `getInfoString(param)` | Generic helper that reads a string property from the OpenCL API. |  
| `getInfoUint(param)` | Reads an unsigned integer property. |  
| `getInfoUint64(param)` | Reads an unsigned 64‑bit property. |  
| `deviceName()` | Returns the device name (`C.CL_DEVICE_NAME`). |  
| `vendorName()` | Returns the vendor name (`C.CL_DEVICE_VENDOR`). |  
| `vendorID()` | Returns the vendor ID (`C.CL_DEVICE_VENDOR_ID`). |  
| `globalMemory()` | Returns the global memory size (`C.CL_DEVICE_GLOBAL_MEM_SIZE`). |  
  
### Main orchestration – `GetGPUDevicesUsingOpenCL`    
```go  
func GetGPUDevicesUsingOpenCL() ([]*sonm.GPUDevice, error) {  
    platforms, err := getPlatforms()  
    ...  
}  
```  
* Iterates over all discovered platforms.    
* For each platform it calls `platform.getGPUDevices()` to obtain a slice of `clDevice`.    
* For every device it pulls the name, vendor name, vendor ID and global memory size, then appends a new `sonm.GPUDevice` struct (from the imported `proto` package) into the result slice.    
* Returns the final list of devices or an error.  
  
### Error handling helper – `errorToString`    
```go  
func errorToString(err C.cl_int) string {  
    switch err { ... }  
}  
```  
* Maps OpenCL return codes to human‑readable strings, covering a wide range of possible errors (success, device not found, compiler failure, etc.).    
  
---  
  
All major parts are now summarized. The file provides a thin wrapper around the OpenCL API that enumerates GPU devices and returns them as Go structs.  
  
# insonmnia/hardware/gpu/cl_other.go  
# Package / Component    
**gpu**  
  
The file defines the `gpu` package which contains a single function that is intended to retrieve GPU devices using OpenCL. The implementation is currently a stub and will need to be filled in later.  
  
---  
  
## Imports  
```go  
import (  
    "github.com/sonm-io/core/proto"  
)  
```  
* `github.com/sonm-io/core/proto` – provides the `GPUDevice` type used as the return value of the function.  
  
---  
  
## External Data / Input Sources    
The function `GetGPUDevicesUsingOpenCL` is expected to query OpenCL for available GPU devices. At present it returns a nil slice and an error placeholder, so the actual data source will be the OpenCL runtime on the host machine.  
  
---  
  
## TODOs  
| # | Item | Notes |  
|---|------|-------|  
| 1 | Implement `GetGPUDevicesUsingOpenCL` to query OpenCL and populate the returned slice. |  
| 2 | Add proper error handling (currently returns `ErrUnsupportedPlatform`). |  
  
---  
  
## Summary of Major Code Parts  
  
### Build Tag  
```go  
// +build !cl  
```  
This file is compiled when the build tag `cl` is **not** set, providing a non‑OpenCL implementation or a fallback.  
  
### Function: `GetGPUDevicesUsingOpenCL`  
*Signature:*    
```go  
func GetGPUDevicesUsingOpenCL() ([]*sonm.GPUDevice, error)  
```  
  
- Currently returns `nil` and an error placeholder.  
- Intended to gather GPU device information via OpenCL and return it as a slice of pointers to `sonm.GPUDevice`.  
- Needs implementation logic for querying the platform, enumerating devices, and handling errors.  
  
---  
  
# insonmnia/hardware/gpu/device.go  
# Package: `gpu`  
  
## Imports    
```go  
import (  
    "errors"  
    "github.com/sonm-io/core/proto"  
)  
```  
* The package uses the standard library `errors` for error handling and imports the `proto` package from `github.com/sonm-io/core`, which provides the `GPUDevice` type used in this file.  
  
## External Data / Input Sources    
* **GetGPUDevicesUsingOpenCL** – a function that must be defined elsewhere in the same package. It is called by `GetGPUDevices` to retrieve the raw list of GPU devices from an OpenCL backend.  
  
## TODOs    
No explicit TODO comments are present in this file, but future work could include:  
1. Fix the misspelled variable name (`devices`) and ensure consistency.  
2. Add error handling for the case where `GetGPUDevicesUsingOpenCL` returns a nil slice or an error.  
  
## Summary of Major Code Parts    
  
### 1. Variable Declaration    
```go  
var (  
    ErrUnsupportedPlatform = errors.New("the platform is not currently supported to expose GPU devices")  
)  
```  
* Declares a package‑level error value that can be returned when the current platform does not support exposing GPU devices.  
  
### 2. Function `GetGPUDevices`    
```go  
func GetGPUDevices() ([]*sonm.GPUDevice, error) {  
    devices, err := GetGPUDevicesUsingOpenCL()  
    if err != nil {  
        return nil, err  
    }  
  
    return devices, nil  
}  
```  
* **Purpose** – Returns a slice of pointers to `sonm.GPUDevice` objects and an error value.    
* **Logic Flow**    
  * Calls the helper `GetGPUDevicesUsingOpenCL()` to obtain the raw device list.    
  * Checks for an error; if one occurs, it propagates that error upward.    
  * Returns the obtained slice (`devices`) unchanged (after correcting the variable name typo).    
  
The function is a thin wrapper around the OpenCL backend call and provides a convenient API for other parts of the `gpu` package to consume GPU device information.  
  
