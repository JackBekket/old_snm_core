# CPU Device Package

## Short Summary  
The `cpu` package exposes a single helper function, **`GetCPUDevice()`**, that gathers low‑level CPU information from the host system using the *gopsutil* library and stores it in an application‑specific struct (`sonm.CPUDevice`). The function reads all available CPU descriptors, extracts the model name, counts sockets, aggregates core counts, and returns a fully populated device object.

---

## Environment Variables, Flags & CLI Arguments  
| Variable / Flag | Purpose |
|------------------|---------|
| **None** | The current implementation does not rely on any external environment variable or command‑line flag. It can be invoked directly from other packages (e.g., `main.go`) with no additional configuration needed. |

---

## File Structure  

```
insonmnia/
└─ hardware/
   └─ cpu/
      └─ device.go
```

*`device.go`* – contains the entire logic for CPU discovery and struct population.

---

## How the Code Works  
1. **Imports**  
   - `errors`: standard error handling.  
   - `github.com/shirou/gopsutil/cpu`: provides `cpu.Info()` that returns a slice of CPU descriptors.  
   - `github.com/sonm-io/core/proto`: supplies the `CPUDevice` struct used to store gathered data.

2. **Function `GetCPUDevice()`**  
   *Signature* – `func GetCPUDevice() (*sonm.CPUDevice, error)`  
   *Logic Flow*  
   - Calls `cpu.Info()`; if it fails, returns the error immediately.  
   - Checks that at least one descriptor was returned (`len(info) == 0` → error).  
   - Uses the first element of the slice to set `ModelName`.  
   - Determines the number of sockets from the length of the slice.  
   - Iterates over all descriptors, summing their core counts into `dev.Cores`.  
   - Returns a pointer to the fully populated device struct and a nil error.

3. **Relations Between Code Entities**  
   * `cpu.Info()` → raw data source → `info` slice.  
   * First element of `info` → `ModelName`.  
   * Length of `info` → number of sockets (`dev.Sockets`).  
   * Loop over `info` → aggregate core counts into `dev.Cores`.

4. **Edge Cases & Launch Scenarios**  
   - If the host system reports no CPU descriptors, the function returns an error `"no CPU detected"`.  
   - The package can be used as a library in a CLI application: e.g., in `main.go` you could call `dev, err := cpu.GetCPUDevice()` and then print or persist the result.  
   - Because the function returns a pointer, callers may modify the returned struct directly before further processing.

---

## Summary of Logic  
- **Input** – CPU descriptors from *gopsutil*.  
- **Processing** – extract model name, count sockets, sum cores.  
- **Output** – `*sonm.CPUDevice` ready for use by other parts of the system.  

This completes the logic of the entire package.