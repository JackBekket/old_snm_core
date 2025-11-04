# disk

## Overview  
The `disk` package provides a single helper function that reports the total and free disk space available for Docker’s root directory. It gathers information from the Docker daemon, queries the filesystem statistics of the root path, and returns those metrics in an `Info` struct.

---

## File structure  

```
insonmnia/hardware/disk/
└── disk.go
```

* **disk.go** – contains all logic for this package.

---

## Imports  
```go
import (
	"context"
	"fmt"
	"syscall"

	"github.com/docker/docker/client"
)
```
| Import | Purpose |
|--------|---------|
| `context` | Provides the `Context` type used in the function signature. |
| `fmt` | Used for error formatting and printing. |
| `syscall` | Calls `Statfs` to obtain filesystem statistics. |
| `github.com/docker/docker/client` | Docker client library that supplies container information. |

---

## External data / configuration sources  

| Source | Description |
|--------|-------------|
| **Environment variables** – `client.NewEnvClient()` reads Docker‑related env vars (e.g., `DOCKER_HOST`, `DOCKER_TLS_VERIFY`) to create a client instance. |
| **Command‑line arguments** – none are required by the current API; the function only needs a `context.Context`. |
| **Flags** – none defined in this file, but the package can be extended with flags that influence the path or output format. |

---

## Major code parts  

### 1. Struct definition – `Info`  
```go
type Info struct {
	TotalBytes uint64
	FreeBytes  uint64
}
```
* Holds two metrics: total disk space and free space, both expressed in bytes.
* Exported fields allow other packages to consume the data.

### 2. Function – `FreeDiskSpace`  
```go
func FreeDiskSpace(ctx context.Context) (*Info, error)
```
* **Purpose**: Compute the amount of free disk space available for Docker’s root directory.
* **Parameters**:  
  * `ctx` – a Go context used to propagate cancellation and deadlines.  

#### Execution flow

1. **Create Docker client**  
   ```go
   cli, err := client.NewEnvClient()
   ```
   Handles errors by returning a formatted message.

2. **Deferred cleanup** – `defer cli.Close()` ensures the client is closed when the function exits.

3. **Retrieve Docker daemon info**  
   ```go
   info, err := cli.Info(ctx)
   ```
   Extracts the root directory path (`info.DockerRootDir`).

4. **Filesystem statistics**  
   * Calls `syscall.Statfs(path, &stat)` to gather block and size information for the Docker root path.
   * If that fails, falls back to the system root `/`.

5. **Return computed values** – The function returns an `Info` instance where:
   * `TotalBytes = stat.Blocks * uint64(stat.Bsize)`
   * `FreeBytes  = stat.Bavail * uint64(stat.Bsize)`

---

## How the package can be used  

* **As a library** – other packages (e.g., monitoring or reporting tools) can import `"insonmnia/hardware/disk"` and call `disk.FreeDiskSpace(ctx)` to obtain disk metrics.
* **As a CLI/command** – if this package is built into a main binary, the entry point could be:
  ```go
  func main() {
      ctx := context.Background()
      info, err := disk.FreeDiskSpace(ctx)
      if err != nil { log.Fatalf("disk: %v", err) }
      fmt.Printf("Total: %d bytes, Free: %d bytes\n", info.TotalBytes, info.FreeBytes)
  }
  ```
  Edge cases:
  * The Docker client may fail to connect – the function returns an error that should be handled.
  * If `syscall.Statfs` fails on the root path, it falls back to `/`; this is a graceful fallback.

---

## Relations between code entities  

* `Info` is returned by `FreeDiskSpace`, so any consumer must import the package and use the struct directly.  
* The function relies on two external calls: `client.NewEnvClient()` for Docker info and `syscall.Statfs` for filesystem stats; both are wrapped in a single error‑handling flow.

---

## Summary  

The `disk` package offers a concise helper that reads Docker’s root directory, queries the underlying filesystem statistics, and returns those metrics as an `Info` struct. It can be used by other components or run directly from a CLI binary to report disk usage for Docker.