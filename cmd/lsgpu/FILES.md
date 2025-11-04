# cmd/lsgpu/main.go  
## Package: `main`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O.  
*   `os`: For operating system functionalities (e.g., exiting the program).  
*   `github.com/sonm-io/core/insonmnia/worker/gpu`: For GPU device collection and metrics retrieval.  
  
**External Data/Input Sources:**  
  
*   `appVersion` (string): A global variable likely set during build time, representing the application's version.  
*   GPU devices: The program interacts with the system's GPU devices to collect metrics.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
---  
  
### Code Summary  
  
The `main` package's primary function is to collect and display information about GPU devices present in the system. It uses the `github.com/sonm-io/core/insonmnia/worker/gpu` package to enumerate and retrieve metrics from these devices.  
  
The program first prints its version (`appVersion`). Then, it calls `gpu.CollectDRICardDevices()` to obtain a list of GPU devices. If this fails, the program exits with an error message.  
  
For each detected GPU device, the program prints its path, temperature, fan speed, power consumption (if available), vendor ID, device ID, major/minor versions, PCI bus ID, and a list of related devices. If metrics retrieval fails for a device, it prints a "metrics is not available" message.  
  
The code is a simple utility for inspecting GPU hardware and its basic operational parameters. It relies on the external `github.com/sonm-io/core/insonmnia/worker/gpu` package for device interaction.  
  
