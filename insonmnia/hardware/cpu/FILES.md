# insonmnia/hardware/cpu/device.go  
## CPU Package Component Summary  
  
**Package Name:** `cpu`  
  
**Imports:**  
*   `errors`: Standard library for error handling.  
*   `github.com/shirou/gopsutil/cpu`: External dependency to retrieve CPU information using gopsutil.  
*   `github.com/sonm-io/core/proto`: Internal package containing the `sonm.CPUDevice` struct definition.  
  
**External Data / Input Sources:**  
*   System's CPU information retrieved via `github.com/shirou/gopsutil/cpu`. The function relies on this external dependency to gather hardware details.  
  
**TODO Comments:** None present in the provided code snippet.  
  
### Code Summary: `GetCPUDevice` Function  
  
The `GetCPUDevice` function retrieves CPU device information and returns it as a `sonm.CPUDevice` struct. It uses gopsutil's `cpu.Info()` to gather details about all detected CPUs on the system. The code assumes that multi-CPU boards have similar CPUs in each socket, so it picks up the model name of the first CPU found and aggregates core counts from all detected CPUs into a single device representation. If no CPUs are detected, an error is returned.  
  
The function returns `sonm.CPUDevice` with fields:  
*   `ModelName`: The model name of the first detected CPU.  
*   `Sockets`: Total number of CPU sockets (length of `info`).  
*   `Cores`: Sum of cores across all CPUs.  
  
