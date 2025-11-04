# lsgpu

A lightweight command‑line tool that collects and prints information about GPU devices on a system.  
The binary is built from the single source file `cmd/lsgpu/main.go` and relies on the external package
`github.com/sonm-io/core/insonmnia/worker/gpu`.

---

## 1. Environment variables, flags & command‑line arguments

| Variable / Flag | Purpose |
|------------------|---------|
| `appVersion` (global) | Holds a string that is printed in the banner line when the program starts. No external configuration file or flag is required; the value can be set at build time via `-ldflags`. |

The binary accepts no command‑line arguments – it simply prints all discovered GPU cards and their metrics.

---

## 2. Project package structure

```
cmd/
└─ lsgpu/
   └─ main.go
```

* `main.go` is the sole source file for the `lsgpu` command package.

---

## 3. Code flow & key entities

1. **Imports**  
   ```go
   import (
       "fmt"
       "os"

       "github.com/sonm-io/core/insonmnia/worker/gpu"
   )
   ```
   * `fmt` – for console output.  
   * `os` – for exit handling.  
   * `gpu` – provides the `CollectDRICardDevices()` function that returns a slice of GPU card descriptors.

2. **Global variable**  
   ```go
   var appVersion string
   ```
   Holds the version string used in the banner line.

3. **Main entry point**  
   ```go
   func main() {
       fmt.Printf("sonm lspgu %s\r\n", appVersion)
       ...
   }
   ```
   * Prints a header with the current `appVersion`.  
   * Calls `gpu.CollectDRICardDevices()` to obtain all GPU card descriptors; if an error occurs, it prints an error message and exits.

4. **Iterating over collected cards**  
   ```go
   for _, card := range cards {
       fmt.Printf("Card: %s\r\n", card.Path)
       ...
   }
   ```
   * For each card in the slice:
     - Prints the device path.
     - Calls `card.Metrics()` to fetch temperature, fan speed and power; prints them if available.
     - Outputs vendor/device IDs, major/minor revisions, PCI bus ID.
     - Loops over `card.Devices` printing each related device string.

5. **Error handling**  
   ```go
   if err != nil {
       fmt.Printf("cannot collect card devces: %v\r\n", err)
       os.Exit(1)
   }
   ```
   * If the call to `CollectDRICardDevices()` fails, an error message is printed and the program terminates with exit code 1.

---

## 4. Edge cases & launch options

| Launch scenario | Command |
|-----------------|---------|
| Build binary locally | `go build -o lsgpu ./cmd/lsgpu` – produces a standalone executable named *lsgpu*. |
| Run directly from source | `go run ./cmd/lsgpu` – executes the command without building an intermediate binary. |
| Use environment variable for version | Set `appVersion` at link time: `go build -ldflags "-X main.appVersion=1.0" -o lsgpu ./cmd/lsgpu`. |

The program currently has no flags or configuration files; all output is driven by the global `appVersion` and the data returned from the external GPU worker package.

---

## 5. Summary

* The `lsgpu` command gathers a list of GPU cards via `gpu.CollectDRICardDevices()`.  
* It prints each card’s path, metrics (temperature, fan speed, power), vendor/device IDs, revision numbers, PCI bus ID and any related devices.  
* All logic resides in a single file; the code is straightforward with no obvious dead branches.  
* Future enhancements could add command‑line flags for filtering or output formatting, or persist the collected data to a file.