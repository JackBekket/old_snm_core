# insonmnia/worker/volume/btfs.go  
# Package / Component    
**File:** `volume/driver.go` (package **volume**)    
  
## Imports  
```go  
import (  
    "context"  
    "fmt"  
    "os"  
    "path/filepath"  
    "sync"  
  
    "bazil.org/fuse"  
    "github.com/docker/docker/api/types"  
    "github.com/docker/docker/api/types/container"  
    "github.com/docker/docker/api/types/mount"  
    "github.com/docker/docker/api/types/network"  
    docker "github.com/docker/docker/client"  
    "github.com/docker/go-plugins-helpers/volume"  
    "github.com/sonm-io/core/util/xdocker"  
    "go.uber.org/zap"  
)  
```  
The file pulls in standard packages for context handling, formatting, OS interaction, path resolution and synchronization, plus Docker‑specific types (container, mount, network), the FUSE helper, a local volume helper, an XDocker utility, and Uber’s Zap logger.  
  
## Constants & Types  
| Constant | Value |  
|----------|-------|  
| `BTFSDriverName` | `"btfs"` |  
| `BTFSImage` | Docker image reference for BTFS (`sonm/btfs@sha256:…`) |  
| `DefaultDockerRootDirectory` | From the local volume helper |  
  
### Types  
* **BTFSDriver** – holds a server pointer and a logger.  
* **BTFSVolume** – simple map of options.  
* **BTFSDockerVolume** – Docker‑specific data (client, mount point, network info, etc.).  
* **BTFSDockerDriver** – the core driver implementation with a mutex, a map of volumes, and a logger.  
  
## BTFSDriver  
### `NewBTFSDriver`  
Creates a new volume server, pulls the BTFS image into Docker, starts the server goroutine, and returns a configured driver instance.  
  
### `CreateVolume` / `RemoveVolume` / `Close`  
* `CreateVolume` logs the request and returns a new `BTFSVolume`.  
* `RemoveVolume` simply logs removal (no real logic yet).  
* `Close` forwards to the underlying server’s close method.  
  
## BTFSVolume  
### `Configure`  
Appends a Docker mount configuration for the volume, setting type, source/target paths, read‑only flag, and driver options. It also configures bind and volume options for the container.  
  
## Helper: `pullImage`  
Pulls the BTFS image from Docker if it isn’t already present, then decodes the pull body with XDocker utilities.  
  
## BTFSDockerDriver  
### Constructor – `NewBTFSDockerDriver`  
* Pulls the BTFS image.  
* Builds a driver instance with a mount root directory under the default Docker root and an empty map of volumes.  
  
### Methods  
  
| Method | Purpose |  
|--------|---------|  
| **Create** | Handles a volume creation request, extracts the magnet URI from options, creates a `BTFSDockerVolume`, and stores it in the internal map. |  
| **List** | Returns all known volumes as a list response. |  
| **Get** | Retrieves a single volume by name. |  
| **Remove** | Deletes a volume from the internal map. |  
| **Path** | Provides the mount point for a given volume. |  
| **Mount** | Creates and starts a Docker container that runs BTFS, mounts the volume into `/root/mnt`, configures networking if supplied, then increments connection count. |  
| **Unmount** | Stops and removes the container, unmounts FUSE, and decrements connections. |  
| **Capabilities** | Declares local scope capability for the driver. |  
  
All methods use a mutex to guard concurrent access, log debug messages with Zap, and rely on Docker client calls (`ContainerCreate`, `ContainerStart`, etc.) to orchestrate containers.  
  
## TODO list  
No explicit `TODO` comments were found in this file.  
  
---  
  
This summary captures the key components of the BTFS volume driver implementation.  
  
# insonmnia/worker/volume/cifs.go  
# Package / Component    
**`volume`** – a Docker plugin that implements a CIFS (Common Internet File System) volume driver.    
  
## Imports    
| Import | Purpose |  
|--------|---------|  
| `context` | Provides the context type used for logging and goroutine handling. |  
| `fmt` | String formatting for options. |  
| `net` | Listener interface for Unix sockets. |  
| `path/filepath` | Path manipulation (joining directories). |  
| `syscall` | System call to obtain GID for socket creation. |  
| `github.com/ContainX/docker-volume-netshare/netshare/drivers` | Driver definitions and constants (`CifsDriver`, `CIFS`). |  
| `github.com/docker/docker/api/types/container` | Docker container configuration types. |  
| `github.com/docker/docker/api/types/mount` | Mount type definitions for Docker volumes. |  
| `github.com/docker/go-connections/sockets` | Unix socket creation helper. |  
| `github.com/docker/go-plugins-helpers/volume` | Generic volume driver interface and handler. |  
| `log "github.com/noxiouz/zapctx/ctxlog"` | Context‑aware logging wrapper. |  
| `go.uber.org/zap` | Structured logger used throughout the file. |  
  
## External Data / Input Sources    
* **Docker root directory** – `volume.DefaultDockerRootDirectory`.    
* **Driver name** – `drivers.CIFS.String()`.    
* **Socket directory** – supplied via options (`opts.socketDir`).    
* **CIFS driver options** – map keys such as `"vers"` for CIFS version.    
  
## TODOs    
No explicit `TODO:` comments were found in the file.  
  
---  
  
# Summary of Major Code Parts  
  
## 1. `cifsVolumeDriver` struct    
A concrete implementation that embeds a generic `drivers.CifsDriver`, holds a Unix socket listener, and a logger. It satisfies the `volume.VolumeDriver` interface expected by Docker’s plugin system.  
  
## 2. `NewCIFSVolumeDriver`    
* **Purpose** – Construct and start a new CIFS volume driver.    
* **Key steps**    
  * Parse optional configuration via variadic `Option`s.    
  * Build root directory path (`baseDir/driverName`).    
  * Resolve socket file path with `fullSocketPath`.    
  * Create a Unix listener using `sockets.NewUnixSocket`.    
  * Instantiate the underlying driver (`drivers.NewCIFSDriver`) and wrap it in a generic handler.    
  * Launch a goroutine that logs initialization and serves the listener.    
* **Return** – A pointer to the initialized `cifsVolumeDriver` instance.  
  
## 3. `CreateVolume` method    
* **Purpose** – Create a new volume within the driver’s namespace.    
* **Key steps**    
  * Log creation request.    
  * Merge optional CIFS version into options map (`drivers.CifsOpts`).    
  * Build a `volume.CreateRequest`.    
  * Call the embedded driver’s `Create` method and return a new `cifsVolume` instance.  
  
## 4. `RemoveVolume` method    
* **Purpose** – Delete an existing volume by name.    
* **Key steps**    
  * Log removal request.    
  * Build a `volume.RemoveRequest`.    
  * Call the embedded driver’s `Remove` method and return any error.  
  
## 5. `Close` method    
* **Purpose** – Gracefully shut down the driver.    
* **Key steps**    
  * Log shutdown message.    
  * Close the Unix listener.  
  
## 6. `cifsVolume` struct & `Configure` method    
* **Purpose** – Represent a single volume and configure its mount settings for Docker containers.    
* **Key steps**    
  * Append a new `mount.Mount` to the container’s host configuration: type, source/target paths, read‑only flag, consistency, and driver options (including CIFS driver name).    
  * Return any error from configuration.  
  
---  
  
All of these parts together provide a functional Docker volume plugin that can create, remove, and configure CIFS volumes via Unix sockets.  
  
# insonmnia/worker/volume/driver.go  
**Package/Component:** `volume`  
  
---  
  
### Imports    
```go  
import (  
	"context"  
	"fmt"  
  
	"github.com/ContainX/docker-volume-netshare/netshare/drivers"  
	"github.com/docker/docker/api/types/container"  
	log "github.com/noxiouz/zapctx/ctxlog"  
	"go.uber.org/zap"  
)  
```  
* `context` – standard Go context handling    
* `fmt` – formatting utilities for error messages    
* `drivers` – local driver registry (CIFS, BTFS)    
* `container` – Docker container host configuration type    
* `zapctx/ctxlog` – contextual logger wrapper (`log`)    
* `go.uber.org/zap` – Zap logging library  
  
---  
  
### Constants    
```go  
const (  
	OptionNetworkName = "NetworkName"  
	OptionNetworkID   = "NetworkID"  
)  
```  
These are keys used when configuring a volume driver.  
  
---  
  
### Types & Interfaces    
  
| Type | Description |  
|------|-------------|  
| `Volume` | Interface for a Docker‑mounted volume. Provides `Configure(mount Mount, cfg *container.HostConfig) error`. |  
| `nilVolume` | Empty implementation of `Volume`, returning nil on `Configure`. |  
| `VolumeDriver` | Interface that creates/removes/cleans up volumes. Methods: `CreateVolume(name string, options map[string]string) (Volume, error)`, `RemoveVolume(name string) error`, `Close() error`. |  
| `nilVolumeDriver` | Empty implementation of `VolumeDriver`. |  
  
---  
  
### Implementations    
  
* **`nilVolume.Configure`** – trivial stub that returns nil.    
* **`nilVolumeDriver.CreateVolume`** – creates a new `nilVolume`.    
* **`nilVolumeDriver.RemoveVolume`** – no‑op removal.    
* **`nilVolumeDriver.Close`** – no‑op close.  
  
---  
  
### Constructors    
  
| Function | Purpose |  
|----------|---------|  
| `NewNilVolumeDriver()` | Returns an instance of the empty driver (`&nilVolumeDriver{}`). |  
| `NewVolumeDriver(ctx context.Context, ty string, options ...Option)` | Factory that creates a concrete driver based on the supplied type. It logs the chosen driver and dispatches to:    
  * `drivers.CIFS.String()` → `NewCIFSVolumeDriver`    
  * `BTFSDriverName` → `NewBTFSDriver`    
  
If an unknown type is passed, it returns an error.  
  
---  
  
### TODOs    
No explicit TODO comments are present in the current file. Future work could include:    
* Implement real logic for `nilVolume.Configure`.    
* Add proper error handling and logging to `CreateVolume`, `RemoveVolume`, and `Close`.    
  
---   
  
**Summary of major code parts**  
  
1. **Imports & constants** – bring in required packages and define option keys.    
2. **Interfaces (`Volume`, `VolumeDriver`)** – declare the contract for volume operations.    
3. **Stub implementations** – provide minimal concrete types that satisfy the interfaces.    
4. **Factory functions** – expose constructors for both a nil driver and a typed driver based on configuration.  
  
These components together form the foundation of the *volume* package, enabling Docker container volume management via pluggable drivers.  
  
# insonmnia/worker/volume/mount.go  
## Package / Component    
**Name:** `volume`    
  
### Imports  
```go  
import (  
    "fmt"  
    "strings"  
)  
```  
The package uses the standard library packages **fmt** for formatting and error handling, and **strings** for string manipulation.  
  
---  
  
## External Data & Input Sources  
- The primary input source is a *specification string* that describes a Docker volume mount in the form    
  `VolumeName:ContainerDestination[:ro]`.    
  Example: `"cifs:/mnt:ro"`.  
- This spec is parsed into up to three parts: source, target and optional permission flag.  
  
---  
  
## TODOs  
No explicit TODO comments are present in this file.    
  
---  
  
## Summary of Major Code Parts  
  
### `Permission` type & constants  
```go  
type Permission uint32  
  
const (  
    RW Permission = iota  
    RO  
)  
```  
Defines a custom unsigned integer type for mount permissions, with two named values: **RW** (read‑write) and **RO** (read‑only).  
  
---  
  
### `ParsePermission(mode string)`    
Parses the permission flag from the spec string.  
- Accepts `"rw"` or `"ro"`.  
- Returns the corresponding `Permission` value and an error if the mode is unknown.  
  
---  
  
### `NewMount(spec string)`    
Creates a `Mount` struct from the Docker‑style specification.  
1. Calls `parseSpec` to split the input into parts.  
2. Handles three cases based on the number of parts:  
   - **1 part**: only target is set.  
   - **2 parts**: source and target are set.  
   - **3 parts**: source, target, and permission flag are set (parsed via `ParsePermission`).  
3. Returns the constructed `Mount` or an error if parsing fails.  
  
---  
  
### `ReadOnly()` method  
```go  
func (m Mount) ReadOnly() bool {  
    return m.Permission == RO  
}  
```  
Convenience helper that reports whether a mount is read‑only.  
  
---  
  
### `Mount` struct definition  
```go  
type Mount struct {  
    Source     string  
    Target     string  
    Permission Permission  
}  
```  
Represents the configuration of a Docker volume mount: source path, target container path, and permission mode.  
  
---  
  
### Helper functions  
  
#### `parseSpec(spec string)`  
Splits the spec into up to three parts using `strings.SplitN`.    
Validates that there are at most two colons and that the first part is non‑empty. Returns an array of strings or an error.  
  
#### `errInvalidSpec(spec string) error`  
Creates a formatted error message for invalid specifications.  
  
---  
  
All functions return errors in case of failure, allowing callers to handle parsing issues gracefully.  
  
# insonmnia/worker/volume/mount_test.go  
**Package / Component**    
`volume`  
  
---  
  
### Imports  
```go  
import (  
	"testing"  
  
	"github.com/stretchr/testify/assert"  
	"github.com/stretchr/testify/require"  
)  
```  
* `testing` – standard Go testing package for unit tests.    
* `github.com/stretchr/testify/assert` – assertion helpers.    
* `github.com/stretchr/testify/require` – requirement helpers (ensures no error).  
  
---  
  
### External Data / Input Sources  
The test file exercises two public functions that are expected to exist in the same package:  
1. **ParsePermission** – parses a permission string (`"rw"` or `"ro"`) into an internal enum value.  
2. **NewMount** – creates a `Mount` struct from a specification string such as `"cifs:/mnt:rw"`.  
  
The tests provide concrete expectations for these functions, so the file serves as both documentation and verification of the parsing logic.  
  
---  
  
### TODOs  
No explicit `TODO:` comments are present in this file.    
(If future work is needed, add a section here.)  
  
---  
  
## Summary of Major Code Parts  
  
| Test | Purpose | Key Assertions |  
|------|---------|----------------|  
| **TestParsePermissionRW** | Verify that `"rw"` parses to the enum value `RW`. | *`assert.Equal(t, RW, perm)`* – checks returned permission. <br>*`require.NoError(t, err)`* – ensures no error was returned. |  
| **TestParsePermissionRO** | Same as above but for `"ro"`. | Checks that parsing yields `RO`. |  
| **TestParsePermissionError** | Ensure an invalid string (`"??"`) returns an error. | *`assert.Error(t, err)`* – verifies an error is produced. |  
| **TestNewMount** | Test full specification with source, target and permission: `"cifs:/mnt:rw"`. | Asserts that the resulting `Mount{Source:"cifs", Target:"/mnt", Permission:RW}` matches expectations and that `ReadOnly()` returns false. |  
| **TestNewMountWithoutPerm** | Same as above but without explicit permission – default should be `RW`. | Checks that omitting the permission still yields a valid mount. |  
| **TestNewMountOnlyTarget** | Test minimal spec `" /mnt"` – only target provided, source defaults to empty string. | Validates that the parser correctly handles missing source and defaults permission to `RW`. |  
| **TestNewMountInvalidSpec** | Verify error handling for an invalid specification (`"whatever:cifs:/mnt:rw"`). | Asserts that a malformed spec results in an empty mount struct and an error. |  
| **TestNewMountInvalidSpecEmptySource** | Test case where source is missing but colon present (`":/mnt:rw"`). | Ensures the parser still returns an error (expected to be handled by `require.NoError`). |  
| **TestNewMountInvalidPerm** | Verify that an unknown permission string (`"cifs:/mnt:?")` results in an error. | Checks that parsing fails gracefully when permission is not recognized. |  
  
---  
  
These tests collectively validate the core functionality of the `volume` package: parsing permissions and constructing mount specifications from strings. They also serve as a reference for expected behavior, which can be used to generate documentation or further test coverage for the entire package.  
  
# insonmnia/worker/volume/options.go  
# Package: **volume**  
  
## Imports    
| Import | Purpose |  
|--------|---------|  
| `fmt` | Standard Go formatting utilities, used for error handling in option constructors. |  
| `go.uber.org/zap` | Uber’s Zap logging library; provides a SugaredLogger that is stored in the options struct. |  
  
---  
  
## External data / input sources    
* **defaultPluginSockDir** – a package‑level constant (defined elsewhere) that supplies the default directory for Unix sockets.    
* **zap.NewNop().Sugar()** – creates a no‑op logger used as the initial value of `log` in the options struct.  
  
---  
  
## TODOs    
No explicit `TODO:` comments are present in this file, but the following items could be considered for future work:    
  
1. Expand `WithOptions(opts map[string]string)` to actually apply the provided key/value pairs to the plugin configuration (currently it only validates the option type).    
  
---  
  
## Summary of major code parts  
  
### 1. `Option` type    
```go  
type Option func(options interface{}) error  
```  
*Defines a functional option that can be applied to an arbitrary options holder. The function receives an empty interface, casts it to the concrete `*options` type, and returns an error if the cast fails.*  
  
### 2. `options` struct    
```go  
type options struct {  
    socketDir string  
    log       *zap.SugaredLogger  
}  
```  
*Holds configuration values for a Docker plugin: the directory where Unix sockets live (`socketDir`) and a SugaredLogger instance (`log`).*  
  
### 3. `newOptions()` function    
```go  
func newOptions() *options {  
    return &options{  
        socketDir: defaultPluginSockDir,  
        log:       zap.NewNop().Sugar(),  
    }  
}  
```  
*Creates a fresh options struct with sensible defaults – the plugin socket directory is set to `defaultPluginSockDir`, and the logger is initialized to a no‑op SugaredLogger.*  
  
### 4. `WithPluginSocketDir(path string)`    
```go  
func WithPluginSocketDir(path string) Option {  
    return func(o interface{}) error {  
        option, ok := o.(*options)  
        if !ok {  
            return fmt.Errorf("invalid option type: %T", o)  
        }  
  
        option.socketDir = path  
        return nil  
    }  
}  
```  
*Produces an `Option` that updates the `socketDir` field of a given options struct. It performs a type assertion, logs an error if the cast fails, and writes the supplied path.*  
  
### 5. `WithLogger(log *zap.SugaredLogger)`    
```go  
func WithLogger(log *zap.SugaredLogger) Option {  
    return func(o interface{}) error {  
        option, ok := o.(*options)  
        if !ok {  
            return fmt.Errorf("invalid option type: %T", o)  
        }  
  
        option.log = log  
        return nil  
    }  
}  
```  
*Similar to `WithPluginSocketDir`, this constructor updates the logger field of an options struct.*  
  
### 6. `WithOptions(opts map[string]string)`    
```go  
func WithOptions(opts map[string]string) Option {  
    return func(o interface{}) error {  
        switch o.(type) {  
        case *options:  
            return nil  
        default:  
            return fmt.Errorf("invalid option type: %T", o)  
        }  
    }  
}  
```  
*Placeholder for a future option that forwards arbitrary key/value pairs to the plugin. Currently it only validates that the passed interface is of type `*options`.*  
  
---  
  
# insonmnia/worker/volume/serve.go  
## Package / Component    
**volume**  
  
### Imports    
| Import | Purpose |  
|--------|---------|  
| `fmt` | String formatting and error handling. |  
| `net` | Network listener interface for Unix sockets. |  
| `os` | File system operations (mkdir, etc.). |  
| `path/filepath` | Path manipulation utilities. |  
| `syscall` | System call constants (e.g., Getgid). |  
| `github.com/docker/go-connections/sockets` | Provides a Unix socket listener. |  
| `github.com/docker/go-plugins-helpers/volume` | Docker volume driver interface and handler. |  
| `go.uber.org/zap` | Structured logging via SugaredLogger. |  
  
### Constants    
* `defaultPluginSockDir = "/run/docker/plugins"` – default directory for plugin sockets.  
  
### Types    
```go  
type VolumeServer struct {  
    name     string          // plugin name  
    listener net.Listener    // Unix socket listener  
    log      *zap.SugaredLogger // logger  
}  
```  
  
### Functions    
  
#### `fullSocketPath(dir, address string) (string, error)`    
* Ensures the directory `dir` exists (`os.MkdirAll`).    
* If `address` is an absolute path, returns it unchanged; otherwise joins `dir`, `address+".sock"` and returns that.    
* Returns the full socket file path or an error.  
  
#### `NewVolumeServer(name string, options ...Option) (*VolumeServer, error)`    
* Builds a new `VolumeServer`.    
* Creates an `options` struct via `newOptions()`, applies any provided option functions.    
* Makes sure the socket directory exists (`os.MkdirAll`).    
* Constructs the Unix socket path and creates a listener with `sockets.NewUnixSocket(path, syscall.Getgid())`.    
* Returns a populated `VolumeServer` instance or an error.  
  
#### `Serve(driver volume.Driver) error`    
* Logs that the plugin is being exposed.    
* Defers a log message for when it stops.    
* Delegates serving to `volume.NewHandler(driver).Serve(m.listener)` – starts listening on the Unix socket.  
  
#### `Close() error`    
* Closes the underlying listener (`m.listener.Close()`).  
  
### External Data / Input Sources    
* **Options**: passed into `NewVolumeServer`; expected to configure fields such as `socketDir`, `log`, etc.    
* **Driver**: a `volume.Driver` instance supplied to `Serve`.    
  
### TODOs    
No explicit TODO comments are present in this file.  
  
---  
  
This file defines the core server component for exposing Docker volume plugins over Unix sockets, handling configuration, socket creation, and lifecycle logging.  
  
