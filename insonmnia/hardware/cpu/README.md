# cpu Package Summary

This package retrieves and aggregates CPU information from the system using the `gopsutil/cpu` library to populate a `sonm.CPUDevice` struct. It assumes that multi-CPU systems have similar CPUs in each socket, taking the model name of the first detected CPU as representative for all. The total core count is calculated by summing cores across all detected CPUs.

**Project Package Structure:**

```
insonmnia/hardware/cpu/
├── device.go
```

**Configuration:**

*   No explicit configuration files or environment variables are used. The package relies entirely on the system's CPU information as reported by `gopsutil`.

**Edge Cases / Launch Conditions:**

The application doesn't have any specific launch conditions, it is a library component that can be called from other parts of the larger project. If no CPUs are detected via `gopsutil`, an error will be returned. The behavior on systems with heterogeneous CPU configurations (different models in different sockets) isn't explicitly handled and may lead to inaccurate reporting.

**Relations Between Code Entities:**

The `device.go` file contains a single function, `GetCPUDevice`. This function directly interacts with the external dependency `github.com/shirou/gopsutil/cpu` to gather CPU information. The returned data is structured according to the internal `sonm.CPUDevice` struct defined in `github.com/sonm-io/core/proto`.