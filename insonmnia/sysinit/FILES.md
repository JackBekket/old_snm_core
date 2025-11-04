# insonmnia/sysinit/lsblk.go  
## Package / Component    
**Name:** `sysinit`    
  
### Imports    
| Import | Purpose |  
|--------|---------|  
| `context` | Provides the `Context` type used for command execution. |  
| `encoding/json` | Handles JSON marshaling/unmarshaling of command output. |  
| `fmt` | Used for error formatting and debugging messages. |  
| `os/exec` | Executes external commands (`lsblk`). |  
  
---  
  
## External Data / Input Sources    
* The function runs the system utility **`lsblk`** with flags `--output NAME,FSTYPE --json`.    
  * Output is captured as a byte slice and decoded into Go structs.    
* Input: a `context.Context` value supplied by callers, allowing cancellation or timeout control.  
  
---  
  
## TODOs    
No explicit `TODO:` comments are present in the current file; future enhancements could include error handling improvements or additional fields for block devices.  
  
---  
  
## Summary of Major Code Parts    
  
### 1. Type Definition – `BlockDevice`  
```go  
type BlockDevice struct {  
    Name     string         `json:"name"`  
    FsType   string         `json:"fstype"`  
    Children []*BlockDevice `json:"children"`  
}  
```  
* Represents a block device as reported by `lsblk`.    
* JSON tags map the fields to the keys returned by the command (`name`, `fstype`).    
* The `Children` slice allows nested devices (e.g., partitions) to be represented recursively.  
  
### 2. Function – `ListBlockDevices`  
```go  
func ListBlockDevices(ctx context.Context) ([]*BlockDevice, error)  
```  
* Public API that returns a list of block devices and an error if any step fails.    
* Accepts a `context.Context` for cancellation/timeout support.  
  
#### 2.1 Command Construction & Execution  
```go  
cmd := exec.CommandContext(ctx, "lsblk", "--output", "NAME,FSTYPE", "--json")  
out, err := cmd.Output()  
```  
* Builds the command with required flags and captures its stdout into `out`.    
* Errors from execution are wrapped with a descriptive message.  
  
#### 2.2 JSON Decoding  
```go  
type container struct {  
    BlockDevices []*BlockDevice `json:"blockdevices"`  
}  
result := &container{}  
if err := json.Unmarshal(out, &result); err != nil { … }  
```  
* Defines an inline wrapper type matching the JSON structure returned by `lsblk`.    
* Unmarshals the raw output into a Go value; errors are propagated with context.  
  
#### 2.3 Return Value  
```go  
return result.BlockDevices, nil  
```  
* Extracts the slice of devices from the container and returns it to callers.  
  
---  
  
The file provides a concise wrapper around `lsblk` that can be reused by other components in the package to discover block devices programmatically.  
  
# insonmnia/sysinit/service.go  
# Package `sysinit`  
  
## Imports  
```go  
import (  
	"context"  
	"fmt"  
	"os"  
	"os/exec"  
	"time"  
  
	"github.com/coreos/go-systemd/dbus"  
	"github.com/docker/docker/pkg/mount"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util/action"  
	"go.uber.org/zap"  
)  
```  
The package pulls in the standard context, fmt, os, exec and time packages plus several third‑party libraries:  
* `dbus` – systemd DBus client for unit management  
* `mount` – Docker mount helper  
* `proto` – sonm core protocol types (used for request/response structs)  
* `action` – action queue abstraction  
* `zap` – Uber’s structured logger  
  
## External data sources  
The service operates on the following external types:  
* `sonm.InitMountRequest`  
* `sonm.InitMountResponse`  
  
These are passed to and returned from the `Mount` method.  
  
---  
  
## Config struct  
```go  
type Config struct {  
	Device      string `yaml:"device"`  
	Cipher      string `yaml:"cipher"`  
	FsType      string `yaml:"fs_type"`  
	MountPoint  string `yaml:"mount_point"`  
}  
```  
* **Device** – e.g. `/dev/sda2` (Docker partition)  
* **Cipher** – encryption type for cryptsetup  
* **FsType** – filesystem type to create  
* **MountPoint** – where the encrypted volume will be mounted  
  
---  
  
## InitService  
The core service that orchestrates all actions.  
  
### `NewInitService`  
Creates a new instance with a config and logger.  
  
### `Reset`  
Runs the action queue returned by `makeActions()` and logs any errors.    
It is intended to bring the system back into a known state.  
  
### `Mount`  
* Checks if Docker is already running (`isMounted`).  
* If not, executes the queued actions with a 10‑second timeout.  
* Returns an empty response on success or propagates errors.  
  
### `isMounted`  
Delegates to `isDockerRunning`.  
  
### `isDockerRunning`  
Uses DBus to list units named `"docker.service"` and verifies that it is active & running.    
Returns true only if the unit exists and its state matches expectations.  
  
### `makeActions`  
Builds a slice of actions that will be executed in order:  
1. Create encrypted volume  
2. Create mount point directory  
3. Create filesystem on the mapper device  
4. Mount the mapper to the mount point  
5. Restart Docker service  
  
---  
  
## Action types  
  
| Type | Purpose | Key fields |  
|------|---------|------------|  
| `CreateEncryptedVolumeAction` | Runs `cryptsetup create …` to make an encrypted block device | `Name`, `Password`, `Device`, `Cipher` |  
| `CreateMountPointAction` | Creates the directory for mounting | `MountPoint`, `Perm` |  
| `CreateFileSystemAction` | Builds a filesystem on `/dev/mapper/<name>` | `Name`, `Type`, `Device` |  
| `MountDeviceMapperAction` | Mounts the mapper device to the mount point | `Name`, `MountPoint`, `Type`, `Options` |  
| `StartDockerAction` | Restarts Docker via DBus | – |  
| `FailAction` | Dummy action that always fails (used for testing) | – |  
  
### CreateEncryptedVolumeAction  
* **Execute** – runs cryptsetup with the provided password and device.  
* **Rollback** – removes the created mapper.  
  
### CreateMountPointAction  
* **Execute** – creates a directory with given permissions.  
* **Rollback** – removes it again.  
  
### CreateFileSystemAction  
* **Execute** – lists block devices, finds the target device, then calls `mkfs.<type>` on `/dev/mapper/<name>`.  
* **findDevice** – recursive search for the configured device name.  
* **makeFileSystem** – runs the mkfs command.  
* **target** – helper that returns the mapper path.  
  
### MountDeviceMapperAction  
* **Execute** – mounts the mapper to the mount point using Docker’s mount helper.  
* **Rollback** – unmounts it again.  
* **target** – same helper as above.  
  
### StartDockerAction  
* **Execute** – restarts the `docker.service` unit via DBus and waits for completion.  
* **Rollback** – stops the unit instead (for cleanup).  
  
### FailAction  
A placeholder that always fails; useful for testing error handling.  
  
---  
  
## TODO list  
No explicit TODO markers were found in this file.    
(If future work is needed, add items here.)  
  
---  
  
# insonmnia/sysinit/service_test.go  
# Package / Component    
**Package name:** `sysinit`    
  
## Imports    
```go  
import (  
	"testing"  
  
	"github.com/magiconair/properties/assert"  
	"github.com/stretchr/testify/require"  
)  
```  
* `testing` – standard Go testing package for unit tests.    
* `github.com/magiconair/properties/assert` – assertion helpers used to compare expected and actual values.    
* `github.com/stretchr/testify/require` – provides the `NoError`, `NotNil`, etc., assertions.  
  
## External Data / Input Sources    
The two test functions each create a slice of pointers to `BlockDevice` structs (`devices`).    
Each `BlockDevice` contains:  
- `Name` (string) – device name.    
- `FsType` (string) – filesystem type.    
- Optional `Children` – slice of child devices.  
  
These slices are the input for the method under test: `action.findDevice(devices)`.  
  
## TODOs    
No explicit TODO comments were found in this file.  
  
---  
  
# Summary of Major Code Parts    
  
## 1. TestCreateFileSystemActionFindDevice    
*Purpose:* Verify that `CreateFileSystemAction.findDevice` correctly locates a top‑level device by name.    
*Setup:*    
```go  
action := CreateFileSystemAction{  
	Name:   "td",  
	Type:   "btrfs",  
	Device: "sr0",  
}  
```  
The test defines two root devices (`sda`, `sr0`). The first has several children, but the search target is the second root device itself.    
*Execution:*    
```go  
dev, err := action.findDevice(devices)  
```  
*Assertions:*    
- No error returned.    
- Returned device pointer is not nil.    
- Device equals a new `BlockDevice` with name `"sr0"` and empty `FsType`.    
  
## 2. TestCreateFileSystemActionFindDeviceChild    
*Purpose:* Verify that the same method can find a child device nested under a root device.    
*Setup:* Same action struct, but now searching for `"sda5"`, which is a child of the first root device (`sda`).    
*Execution & Assertions:* Identical to the previous test; the expected result is the child `BlockDevice` with name `"sda5"` and filesystem type `"ext4"`.  
  
Both tests confirm that `findDevice` traverses both the top level slice and any nested children when searching for a device by its name.  
  
