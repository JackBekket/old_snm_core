# ram  

## Overview  
`insonmnia/hardware/ram/device.go` implements a small helper that reads the current system memory statistics and returns them wrapped in a `sonm.RAMDevice` struct. The file contains only one exported function, `NewRAMDevice`, which is intended to be used by other parts of the project or as a CLI entry point.

---

## Package structure  

```
insonmnia/hardware/ram/
├── device.go
```

* **device.go** – main source file; defines package `ram` and provides the public constructor `NewRAMDevice`.

---

## Imports & external data sources  
| Import | Purpose |
|--------|---------|
| `github.com/shirou/gopsutil/mem` | Provides `mem.VirtualMemory()` to read system memory statistics. |
| `github.com/sonm-io/core/proto` | Supplies the `RAMDevice` struct type used as return value. |

The function pulls a `VirtualMemoryStat` from `gopsutil`, then maps its fields into a new `sonm.RAMDevice`.  
*Note:* The field mapping uses `m.Total` for both `Total` and `Available`; this may be intentional or an oversight (perhaps it should use `m.Available`).  

---

## Environment variables, flags & command‑line arguments  
No explicit configuration files or flag parsing are present in the current package. If the project is built as a CLI tool, the only required environment variable would be any that `gopsutil` expects (none by default). The function can be called directly from other packages; no command‑line flags are defined here.

---

## Edge cases & launch scenarios  
* **CLI/entry point** – If this package is imported into a main program, the exported `NewRAMDevice()` can be invoked to obtain current RAM statistics.  
* **Error handling** – The function returns an error if `mem.VirtualMemory()` fails; callers should check for `nil` before using the returned pointer.  

---

## Summary of code logic  

1. **Package declaration**: `package ram`.  
2. **Import block** pulls in memory stats and the core data struct.  
3. **Function `NewRAMDevice`**  
   ```go
   func NewRAMDevice() (*sonm.RAMDevice, error) {
       m, err := mem.VirtualMemory()
       if err != nil { return nil, err }
   
       return &sonm.RAMDevice{
           Total:     m.Total,
           Available: m.Total,  // likely should be m.Available
           Used:      m.Used,
       }, nil
   }
   ```  
   * Calls `mem.VirtualMemory()` to obtain a snapshot of system memory.  
   * Handles any error immediately.  
   * Constructs and returns a pointer to a new `sonm.RAMDevice` with the current values.

---

**End of output**