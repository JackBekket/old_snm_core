# insonmnia/worker/plugin/cleanup.go  
**Package name & imports**    
- **package:** `plugin`    
- **imports:**  
  ```go  
  import (  
      "fmt"  
      "github.com/sonm-io/core/insonmnia/worker/volume"  
  )  
  ```  
  
---  
  
### External data / input sources    
The code relies on the following external type:  
- `volume.VolumeDriver` – used by the `volumeCleanup` struct to remove a volume.  
  
No other external data sources are referenced in this file.  
  
---  
  
### TODO comments    
There are currently no explicit `TODO:` comments in the provided snippet.    
  
---  
  
## Summary of major code parts  
  
#### 1. `Cleanup` interface  
```go  
type Cleanup interface {  
    Close() error  
}  
```  
Defines a contract for cleanup operations that return an `error`. Both `nestedCleanup` and `volumeCleanup` implement this interface.  
  
#### 2. `nestedCleanup` struct & methods    
- **Fields**: `children []Cleanup` – holds child cleanup objects.    
- **Constructor**: `newNestedCleanup()` creates a new instance with an empty slice.    
- **Add(v Cleanup)**: appends a child to the list.    
- **Close() error**: iterates over all children, calls their `Close`, aggregates any returned errors into a slice, resets the struct to a fresh state (`*newNestedCleanup()`), and returns either `nil` or an aggregated error via `fmt.Errorf`.  
  
#### 3. `volumeCleanup` struct & method    
- **Fields**: `driver volume.VolumeDriver` and `id string`.    
- **Close() error**: simply delegates to the underlying driver’s `RemoveVolume` method, passing its stored id.  
  
These components together allow a plugin system to manage cleanup of nested resources and volumes in a unified way.  
  
# insonmnia/worker/plugin/config.go  
**Package name**    
`plugin`  
  
---  
  
### Imports    
```go  
import "github.com/sonm-io/core/insonmnia/worker/network"  
```  
The package pulls in the `network` module which supplies the types used for overlay drivers (`TincNetworkConfig`, `L2TPConfig`).  
  
---  
  
### External data / input sources    
* YAML configuration file – the struct tags indicate that the following keys are expected:  
  * `socket_dir` → value for `SocketDir`  
  * `volume` → nested map for `VolumesConfig`  
  * `overlay` → nested map for `OverlayConfig`  
  
The default values supplied in the tags (`/run/docker/plugins`, `/var/lib/docker-volumes`) provide sensible fall‑backs when a key is omitted.  
  
---  
  
### TODOs    
No explicit `TODO:` comments are present in this snippet, but the structure suggests future work could involve:  
* Validation of GPU map entries  
* Loading and applying overlay driver configurations  
  
---  
  
## Summary of major code parts    
  
#### 1. `Config` struct    
The top‑level configuration container holds four fields:  
  
| Field | Type | Purpose |  
|-------|------|---------|  
| `SocketDir` | `string` | Directory where Docker plugin sockets are stored (default `/run/docker/plugins`). |  
| `Volumes` | `VolumesConfig` | Configuration for Docker volume drivers. |  
| `Overlay` | `OverlayConfig` | Configuration for overlay networking drivers. |  
| `GPUs` | `map[string]map[string]string` | A two‑level map that can hold GPU identifiers and their associated configuration values (e.g., driver name → options). |  
  
#### 2. `VolumesConfig` struct    
Defines the root directory for Docker volumes (`Root`) and a nested map of drivers (`Drivers`). The map allows multiple volume drivers to be configured under distinct keys.  
  
#### 3. `OverlayConfig` struct    
Contains an embedded `Drivers` struct that holds pointers to two overlay driver configs:  
* `Tinc` – configuration for the Tinc network driver.  
* `L2TP` – configuration for the L2TP network driver.  
  
Both are tagged with YAML names (`tinc`, `l2tp`) so they can be populated from a YAML file.  
  
---  
  
This file establishes the data model that will later be read, validated and applied by other components of the plugin package.  
  
# insonmnia/worker/plugin/plugin.go  
**Package / Component**    
`plugin`  
  
### Imports  
```go  
import (  
    "context"  
    "fmt"  
    "sort"  
  
    "github.com/docker/docker/api/types/container"  
    "github.com/docker/docker/api/types/network"  
    "github.com/docker/docker/client"  
    log "github.com/noxiouz/zapctx/ctxlog"  
    "github.com/sonm-io/core/insonmnia/hardware"  
    "github.com/sonm-io/core/insonmnia/structs"  
    "github.com/sonm-io/core/insonmnia/worker/gpu"  
    minet "github.com/sonm-io/core/insonmnia/worker/network"  
    "github.com/sonm-io/core/insonmnia/worker/storage"  
    "github.com/sonm-io/core/insonmnia/worker/volume"  
    "github.com/sonm-io/core/proto"  
    "go.uber.org/zap"  
)  
```  
  
### External data / input sources  
| Source | Description |  
|--------|-------------|  
| `Config` | Configuration for volumes, GPUs and overlay drivers (tinc/l2tp). |  
| `container.HostConfig` | Docker host configuration passed to tuning functions. |  
| `network.NetworkingConfig` | Networking configuration used by network tuners. |  
| `hardware.Hardware` | Hardware state that will be enriched with GPU and network info. |  
  
### TODO comments  
* In **NewRepository** – “NOTE: not sure it's safe to do it here. Please, suggest better place”.  
* In **PostCreationTune** – “NOTE: move it to r.TuneStorageQuota”.  
  
---  
  
## Summary of major code parts  
  
### Repository struct  
```go  
type Repository struct {  
    volumes           map[string]volume.VolumeDriver  
    gpuTuners         map[sonm.GPUVendorType]gpu.Tuner  
    networkTuners     map[string]minet.Tuner  
    storageQuotaTuner storage.StorageQuotaTuner  
}  
```  
Holds all plugin drivers and tuners. Maps are keyed by driver type or network name.  
  
### NewRepository  
* Builds a new `Repository` from a supplied `Config`.  
* Instantiates volume drivers, GPU tuners, and overlay network tuners (tinc/l2tp).  
* If Docker supports quota, creates a storage quota tuner.  
* Returns the fully populated repository or an error.  
  
### EmptyRepository  
Convenience constructor used mainly in tests; returns a repository with empty maps ready for population.  
  
### Tune orchestration  
`Tune(ctx, provider, hostCfg, netCfg)`    
* Logs tuning start.    
* Calls `TuneGPU`, `TuneVolumes`, and `TuneNetworks`.    
* Aggregates cleanup objects into a nested cleanup chain that can be used later to roll back or clean up.  
  
### GPU handling  
* `HasGPU()` – quick check for presence of GPU tuners.  
* `collectGPUDevices()` – gathers all GPU devices from the tuners, sorts them by ID.  
* `ApplyHardwareInfo(hw)` – enriches a `hardware.Hardware` instance with GPU and network flags.  
* `TuneGPU(provider, cfg)` – iterates over GPU tuners to apply GPU settings to the host config.  
  
### Volume handling  
* `TuneVolumes(ctx, provider, cfg)` – creates volumes for each provider entry, sets network options, mounts them, and registers a volume cleanup.  
* `GetVolumeCleaner(ctx, provider)` – prepares clean‑up objects for all provider volumes.  
* `PostCreationTune` – after the main tuning, optionally tunes storage quota if needed.  
  
### Network handling  
* `TuneNetworks(ctx, provider, hostCfg, netCfg)` – iterates over network specs from a provider and applies each tuner.  
* `GetNetworkCleaner(ctx, provider)` – prepares clean‑up objects for all networks.  
* `JoinNetwork(ID)` – finds the first network tuner that has tuned the given ID and returns its invitation spec.  
  
### Close  
Closes all drivers (volume, GPU) and aggregates any errors into a single error return.  
  
---  
  
All functions use a nested cleanup pattern (`newNestedCleanup()`) to allow sequential execution of tuning steps with rollback support. The repository is the central hub that connects configuration data, Docker API clients, and plugin-specific tuners for GPUs, volumes, networks, and storage quota.  
  
