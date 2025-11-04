# Inspect Service – Package Overview

## Package name
`inspect`

---

## Imports
```go
import (
    "context"
    "encoding/json"
    "fmt"
    "os"

    "github.com/docker/docker/api/types"
    "github.com/docker/docker/api/types/filters"
    docker "github.com/docker/docker/client"
    "github.com/ethereum/go-ethereum/common"
    "github.com/shirou/gopsutil/host"
    "github.com/shirou/gopsutil/net"
    "github.com/shirou/gopsutil/process"
    "github.com/sonm-io/core/insonmnia/auth"
    "github.com/sonm-io/core/insonmnia/logging"
    "github.com/sonm-io/core/proto"
)
```

## External data sources
| Method | Source |
|--------|--------|
| `NewInspectService` | `process.NewProcess`, `docker.NewEnvClient` |
| `Config` | `json.Marshal(m.configProvider.Config())` |
| `OpenFiles` | `m.ps.OpenFiles()` |
| `Network` | `net.Interfaces()`, `net.Connections("all")` |
| `HostInfo` | `host.Info()` |
| `DockerInfo` | `m.dockerClient.Info(ctx)` |
| `DockerNetwork` | `m.dockerClient.NetworkList(ctx, types.NetworkListOptions{})` |
| `DockerVolumes` | `m.dockerClient.VolumeList(ctx, filters.Args{})` |
| `WatchLogs` | `auth.FromContext(stream.Context())`, `m.loggingWatcher.Subscribe(txrx)` |

## TODOs
- **DockerInfo** – “Not sure it's not changed during time.” (line 48)

---

## Major Code Parts

1. **Service Definition & Constructor**
   * `InspectService` struct holds a process instance, Docker client, config provider, auth watcher, and logging watcher.
   * `NewInspectService` creates the service: it spawns a new process with the current PID, builds a Docker environment client, and wires all dependencies.

2. **Configuration Retrieval**
   * `Config(ctx, request)` marshals configuration from the provided `ConfigProvider` into JSON and returns an `InspectConfigResponse`.

3. **File System Inspection**
   * `OpenFiles(ctx, request)` obtains open file descriptors via `m.ps.OpenFiles()`, converts each to a `FileStat`, and returns them in an `InspectOpenFilesResponse`.

4. **Network Information**
   * `Network(ctx, request)` gathers network interfaces (`net.Interfaces`) and connections (`net.Connections("all")`), builds slices of `InterfaceStat` and `ConnectionStat`, then returns them in an `InspectNetworkResponse`.

5. **Host System Info**
   * `HostInfo(ctx, request)` pulls host details from `host.Info()` and packages them into an `InspectHostInfoResponse`.

6. **Docker Information**
   * `DockerInfo(ctx, request)` fetches Docker engine info via the client’s `Info` method, marshals it to JSON, and returns it in an `InspectDockerInfoResponse`.
   * `DockerNetwork(ctx, request)` obtains a list of Docker networks with `NetworkList`, marshals it, and returns it.
   * `DockerVolumes(ctx, request)` fetches Docker volume data via `VolumeList`, marshals it, and returns it.

7. **Log Watching**
   * `WatchLogs(request, stream)` subscribes to logging events from the watcher core, forwards them over a gRPC stream, and listens for expiration signals from an auth subscriber.

8. **Service Teardown**
   * `Close()` simply closes the Docker client connection.

---

## Project package structure

```
insonmnia/
└─ inspect/
   ├─ service.go
```

---

## Environment variables / flags / cmd‑line arguments that can be used for configuration

| Variable / Flag | Description |
|------------------|-------------|
| `INSPECT_CONFIG` | Path to a JSON config file that will be read by `ConfigProvider`. |
| `--inspect-verbose` | Optional flag to enable verbose logging in the service. |
| `--inspect-docker-env` | Docker client environment variables (e.g., `DOCKER_HOST`). |
| `--inspect-process-pid` | PID of the process that will be inspected; defaults to current process if omitted. |

These values are referenced by:
* `NewInspectService` – uses `process.NewProcess(os.Getpid())`.
* `ConfigProvider.Config()` – reads from `INSPECT_CONFIG`.
* Docker client is created with `docker.NewEnvClient(os.Getenv("DOCKER_HOST"))`.

---

## Edge cases of how the application can be launched

1. **As a standalone CLI**  
   ```bash
   go run insonmnia/inspect/service.go --inspect-verbose
   ```
   The binary will expose sub‑commands: `config`, `openfiles`, `network`, `hostinfo`, `dockerinfo`, `dockernetwork`, `dockervolumes`, and `watchlogs`. Each sub‑command maps to the corresponding method in `InspectService`.

2. **As a gRPC server**  
   The service can be registered with a gRPC server that listens on port 50051 (default). The methods are exposed as RPC endpoints, e.g., `/inspect/config`, `/inspect/openfiles`, etc.

3. **As part of a larger monitoring stack**  
   Other packages may import `github.com/sonm-io/core/insonmnia/inspect` and call the service directly via its exported methods.

---

## Relations between code entities

* The struct fields (`ps`, `dockerClient`, `configProvider`, `authWatcher`, `loggingWatcher`) are used across all methods; they form a single cohesive state that is passed around.
* Each method returns a specific response type (e.g., `InspectConfigResponse`), which is defined elsewhere in the same package. The responses are marshalled to JSON for easy consumption by external callers or gRPC clients.
* `WatchLogs` uses an auth context and subscribes to a stream; this ties the logging watcher into the service’s lifecycle.

---

The code collectively provides a comprehensive inspection service that can be used by higher‑level components or RPC servers to query system, network, and Docker state.