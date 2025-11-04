# insonmnia/hardware/ram/device.go  
## Package: `ram`  
  
**Imports:**  
  
*   `github.com/shirou/gopsutil/mem`: Used for retrieving system memory information.  
*   `github.com/sonm-io/core/proto`: Used for defining the `sonm.RAMDevice` struct.  
  
**External Data/Input Sources:**  
  
*   System memory statistics obtained via `mem.VirtualMemory()` from the `gopsutil/mem` package. This relies on the underlying operating system's memory reporting mechanisms.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
**Code Summary:**  
  
### `NewRAMDevice()` Function  
  
This function creates a `sonm.RAMDevice` instance populated with system RAM statistics. It retrieves total, available, and used memory using `mem.VirtualMemory()` from the `gopsutil/mem` package. If an error occurs during memory retrieval, the function returns an error. Otherwise, it returns a pointer to a `sonm.RAMDevice` struct initialized with the retrieved memory values. The `Total` and `Available` fields are set to the same value, which might be a simplification or an intended behavior depending on the broader context of the `sonm` package.  
  
