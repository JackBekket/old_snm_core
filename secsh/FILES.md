# secsh/banner.go  
**Package / Component**    
`secsh`  
  
---  
  
### Imports  
| Package | Purpose |  
|---------|---------|  
| `context` | Provides context handling for system calls |  
| `fmt` | Formatting strings |  
| `strings` | String manipulation (Title) |  
| `time` | Time formatting and current time |  
| `github.com/shirou/gopsutil/disk` | Disk usage metrics |  
| `github.com/shirou/gopsutil/host` | Host information & users |  
| `github.com/shirou/gopsutil/load` | Load average |  
| `github.com/shirou/gopsutil/mem` | Memory and swap stats |  
| `go.uber.org/zap` | Structured logging |  
  
---  
  
### External Data Sources  
* **Host Info** – `host.InfoWithContext(ctx)`  
* **Load Average** – `load.AvgWithContext(ctx)`  
* **Disk Usage** – `disk.UsageWithContext(ctx, "/")`  
* **Virtual Memory** – `mem.VirtualMemoryWithContext(ctx)`  
* **Swap Memory** – `mem.SwapMemoryWithContext(ctx)`  
* **Users List** – `host.UsersWithContext(ctx)`  
  
All data are gathered in the constructor `NewBanner` and formatted into a banner string.  
  
---  
  
### TODOs  
No explicit TODO comments exist, but potential improvements could be:  
1. Handle errors more robustly (e.g., return error from `NewBanner`).    
2. Cache or reuse already fetched values to avoid repeated calls if called multiple times.    
  
---  
  
## Code Overview  
  
#### 1. Struct Definition  
```go  
type Banner struct {  
    banner []byte  
}  
```  
* Holds the raw byte slice that represents the banner text.  
  
#### 2. `NewBanner` – Constructor & Data Aggregation  
- Accepts a context and a logger.  
- Calls all external data sources, logs failures with `zap.SugaredLogger`.  
- Builds a human‑readable header line using platform, OS, kernel version, etc.  
- Adds subsequent lines for system load, disk usage, memory usage, swap usage, process count, and logged‑in users.  
- Returns the fully populated `*Banner`.  
  
#### 3. `AddLine` – Helper  
```go  
func (m *Banner) AddLine(line string) {  
    m.banner = append(m.banner, []byte(line)...)  
    m.banner = append(m.banner, '\n')  
}  
```  
Appends a line and a newline to the banner slice.  
  
#### 4. `String` – String Representation  
```go  
func (m *Banner) String() string {  
    return string(m.banner)  
}  
```  
Converts the internal byte slice into a Go string for display or further processing.  
  
---  
  
**<end_of_output>**  
  
# secsh/config.go  
**Package/Component Name**    
`secsh`  
  
---  
  
### Imports  
```go  
import (  
	"github.com/sonm-io/core/accounts"  
	"github.com/sonm-io/core/insonmnia/npp"  
)  
```  
* `github.com/sonm-io/core/accounts` – provides the `accounts.EthConfig` type used in the configuration struct.    
* `github.com/sonm-io/core/insonmnia/npp` – supplies the `npp.Config` type for NPP‑specific settings.  
  
---  
  
### External Data / Input Sources  
The struct fields are annotated with YAML tags, indicating that this configuration is expected to be loaded from a YAML file (or similar source).    
* `secexec` – path to the security execution binary.    
* `seccomp_policy_dir` – directory containing seccomp policy files.    
* `allowed_keys` – list of keys permitted in the configuration.    
* `ethereum` – nested Ethereum configuration (`accounts.EthConfig`).    
* `npp` – nested NPP configuration (`npp.Config`).  
  
---  
  
### TODOs  
No explicit TODO comments are present in this snippet.  
  
---  
  
## Summary of Major Code Parts  
  
#### **1. Package Declaration**  
The file declares the Go package `secsh`, which likely contains logic related to security shell or similar functionality within the larger project.  
  
#### **2. Import Section**  
Two external packages are imported:  
* `accounts` – for Ethereum configuration handling.  
* `npp` – for NPP (possibly “NPP” stands for a specific protocol or module) configuration.  
  
These imports suggest that this file is part of a configuration subsystem that aggregates settings from multiple sub‑packages.  
  
#### **3. Config Struct**  
The core element of the file is the `Config` struct, which aggregates several configuration parameters:  
* `SecExecPath` – string path to an executable; YAML key `secexec`.  
* `SeccompPolicyDir` – directory for seccomp policies; YAML key `seccomp_policy_dir`.  
* `AllowedKeys` – slice of strings specifying allowed keys; YAML key `allowed_keys`.  
* `Eth` – nested Ethereum configuration (`accounts.EthConfig`); YAML key `ethereum`.  
* `NPP` – nested NPP configuration (`npp.Config`); YAML key `npp`.  
  
The struct is designed to be unmarshalled from a YAML source, making it straightforward to read or write configuration files that drive the behavior of the `secsh` package.  
  
---  
  
This file therefore serves as a lightweight data container for configuration values used by the `secsh` component. It pulls in two external types and exposes them via a single struct with clear YAML tags, ready for serialization/deserialization.  
  
# secsh/exec.go  
**Package / Component Name**    
`secsh`  
  
---  
  
## Imports  
```go  
import (  
	"bytes"  
	"io"  
	"os/exec"  
)  
```  
* `bytes` – used to buffer command output/error streams.    
* `io` – provides the pipe and writer interfaces for chaining commands.    
* `os/exec` – contains the `Cmd` type that represents an external process.  
  
---  
  
## External Data / Input Sources  
| Function | Parameters | Description |  
|----------|------------|-------------|  
| `parsePipedCommand` | `args []string` | Parses a slice of command‑line arguments into a two‑dimensional slice where each sub‑slice represents one piped command. |  
| `execPipedCommand` | `out io.Writer`, `cmds ...*exec.Cmd` | Receives an output writer and a variadic list of commands to execute in a pipeline. |  
| `execNext` | `stack []*exec.Cmd`, `pipes []*io.PipeWriter` | Helper that starts the first two commands, then recursively runs the rest while closing pipes as they finish. |  
  
---  
  
## TODOs  
No explicit `TODO:` comments are present in this file.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. `parsePipedCommand`  
* **Purpose** – Convert a flat argument list into a nested slice where each inner slice is a command separated by the pipe (`|`) token.  
* **Logic** – Iterates over `args`, accumulating tokens until it encounters `"|"`. When a pipe is found, the current command slice is appended to the result and reset. After the loop, any remaining tokens are added as the final command.  
* **Return Value** – A two‑dimensional string slice (`[][]string`) representing all piped commands; error always `nil` (no error handling yet).  
  
### 2. `execPipedCommand`  
* **Purpose** – Wire together a series of `exec.Cmd` processes so that the standard output of each command feeds into the standard input of the next.  
* **Setup** – Creates a buffer (`bytes.Buffer`) for capturing stderr, builds an array of pipe writers (`pipes`) sized to one less than the number of commands, and iterates over all but the last command:  
  * `Stdout` of the current command is set to its corresponding stdout pipe.  
  * `Stderr` of the current command writes into the error buffer.  
  * The next command’s stdin is connected to the same pipe.  
* **Final Command** – The last command’s stdout is directed to the supplied writer (`out`) and its stderr also goes to the buffer.  
* **Execution** – Calls `execNext` to start the pipeline; returns any error that propagates from it.  
  
### 3. `execNext`  
* **Purpose** – Recursively launch commands in a pipeline, ensuring each command starts before the next one finishes.  
* **Logic** – Starts the first command (`stack[0]`). If there is more than one command left, it also starts the second and defers a recursive call that will close the current pipe once the first command completes. The function finally waits for the first command to finish and returns any error.  
  
---  
  
This file implements the core plumbing of a simple shell‑like pipeline executor: parsing arguments into commands, wiring them together with pipes, and running them in sequence.  
  
# secsh/server.go  
**Package / Component**    
`secsh`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"crypto/ecdsa"  
	"crypto/tls"  
	"fmt"  
	"time"  
  
	"github.com/ethereum/go-ethereum/common"  
	"github.com/ethereum/go-ethereum/crypto"  
	"github.com/sonm-io/core/insonmnia/auth"  
	"github.com/sonm-io/core/insonmnia/npp"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/secsh/secshc"  
	"github.com/sonm-io/core/util"  
	"github.com/sonm-io/core/util/xgrpc"  
	"go.uber.org/zap"  
	"golang.org/x/sync/errgroup"  
	"google.golang.org/grpc"  
)  
```  
The file pulls in standard Go packages (`context`, `crypto/ecdsa`, `crypto/tls`, `fmt`, `time`) and a set of project‑specific libraries that provide Ethereum utilities, NPP networking, gRPC helpers, logging, and configuration handling.  
  
---  
  
### External data / input sources  
| Source | Description |  
|--------|-------------|  
| `m.cfg` | Holds the server configuration (`SecExecPath`, `SeccompPolicyDir`, `NPP.Backlog`, etc.). |  
| `npp.NewListener` | Creates a TCP listener on address `"127.0.0.1:0"` using the protocol defined in `secshc.Protocol`. |  
| `xgrpc.NewTransportCredentials(tlsConfig)` | TLS credentials for gRPC transport. |  
| `auth.AuthRouter` | Authorization logic built from keys listed in `m.cfg.AllowedKeys`. |  
  
---  
  
### TODOs  
- **runACLUpdateLoop** – the loop currently only logs “updating ACL”; the actual whitelist update from a remote source is yet to be implemented.  
  
---  
  
## Summary of major code parts  
  
#### 1. `execStream` struct & `Write`  
```go  
type execStream struct {  
	Server sonm.RemotePTY_ExecServer  
}  
```  
*Purpose*: Implements a simple writer that sends byte slices as chunks over the gRPC server.    
The `Write` method packages the data into a `RemotePTYExecResponseChunk`, marks it as not done, and forwards it via `m.Server.Send`. It returns the number of bytes written.  
  
#### 2. `RemotePTYServer` struct  
```go  
type RemotePTYServer struct {  
	cfg        *Config  
	privateKey *ecdsa.PrivateKey  
	log        *zap.SugaredLogger  
}  
```  
*Purpose*: Holds runtime state for a remote PTY server: configuration, an ECDSA key used for TLS and Ethereum address derivation, and a logger.  
  
#### 3. `NewRemotePTYServer`  
Creates a new instance of the server by loading an ECDSA key from the config (`cfg.Eth.LoadKey`) and storing it in the struct. It returns the initialized object or an error if key loading fails.  
  
#### 4. `Run` method  
```go  
func (m *RemotePTYServer) Run(ctx context.Context) error { … }  
```  
*Workflow*    
1. Logs the Ethereum address derived from the private key.    
2. Builds a TLS cert rotator (`util.NewHitlessCertRotator`).    
3. Creates an authorization router via `makeAuthorization`.    
4. Instantiates a gRPC server with that authorization and a local service implementation (`RemotePTYService`).    
5. Starts an NPP listener on the loopback address, using the protocol from `secshc` and TLS credentials.    
6. Launches two goroutines: one for ACL updates (`runACLUpdateLoop`) and one to serve gRPC traffic.    
7. Waits until context cancellation, then closes the listener and returns any errors.  
  
#### 5. `makeAuthorization`  
```go  
func (m *RemotePTYServer) makeAuthorization(ctx context.Context) *auth.AuthRouter { … }  
```  
*Purpose*: Builds an authorization chain that combines all keys listed in `cfg.AllowedKeys`.    
It creates a base router (`NewAnyOfTransportCredentialsAuthorization`), adds each key to it, then wraps the whole thing into an event‑based authorizer with fallback logic.  
  
#### 6. `makeServer`  
```go  
func (m *RemotePTYServer) makeServer(ctx context.Context, tlsConfig *tls.Config, authorization *auth.AuthRouter) *grpc.Server { … }  
```  
*Purpose*: Configures a gRPC server using the provided TLS credentials and authorization router.    
It sets up standard interceptors for tracing, logging, and verification before returning the fully configured server.  
  
#### 7. `runACLUpdateLoop`  
```go  
func (m *RemotePTYServer) runACLUpdateLoop(ctx context.Context) error { … }  
```  
*Purpose*: Periodically refreshes the ACL used by the PTY service.    
It currently logs a message every five seconds and contains a TODO placeholder for the actual update logic.  
  
---  
  
All parts together provide a gRPC‑based remote PTY server that can be started, served over NPP, and periodically refreshed.  
  
# secsh/service.go  
**Package / Component**    
- **Name:** `RemotePTYService` (file belongs to package `secsh`)    
  
**Imports**    
```go  
import (  
    "context"  
    "fmt"  
    "io/ioutil"  
    "os/exec"  
    "path"  
    "strings"  
  
    "github.com/sonm-io/core/proto"  
    "go.uber.org/zap"  
)  
```  
  
---  
  
## External data / input sources    
| Source | Type | Description |  
|--------|------|-------------|  
| `sonm.RemotePTYBannerRequest` | request | Input for the banner method |  
| `sonm.RemotePTYBannerResponse` | response | Output of the banner method |  
| `sonm.RemotePTYExecRequest` | request | Input for exec execution |  
| `sonm.RemotePTY_ExecServer` | server | gRPC streaming server used in Exec |  
| `execStream` | custom type (defined elsewhere) | Holds the streaming context |  
  
---  
  
## TODOs    
No explicit `TODO:` comments were found in this file.  
  
---  
  
# Summary of major code parts  
  
### 1. Service struct definition  
```go  
type RemotePTYService struct {  
    execPath   string  
    policyPath string  
    log        *zap.SugaredLogger  
}  
```  
Holds configuration for the PTY service: path to executable, directory containing policies, and a logger.  
  
---  
  
### 2. `Banner` method    
*Purpose:* Build a banner that lists all available commands found in `policyPath`.    
*Key steps:*    
1. Call `commandsList()` to get command names.    
2. Create a new banner via `NewBanner(ctx, m.log)`.    
3. Append an empty line and a formatted list of commands.    
4. Return the banner string wrapped in a `RemotePTYBannerResponse`.  
  
---  
  
### 3. `commandsList` helper    
*Purpose:* Read the directory at `policyPath`, filter for `.yaml` files, and return their base names (without extension).    
*Key steps:*    
1. Use `ioutil.ReadDir` to list entries.    
2. Iterate over each file; skip directories and non‑`.yaml` files.    
3. Strip the `.yaml` suffix from the filename and append it to a slice of strings.    
4. Return that slice.  
  
---  
  
### 4. `Exec` method    
*Purpose:* Entry point for executing a PTY command.    
*Key steps:*    
1. Call `execCmd` with the request arguments and an `execStream` wrapper around the gRPC server.  
  
---  
  
### 5. `execCmd` helper    
*Purpose:* Parse piped commands, prepare arguments, build exec.Cmd objects, and run them in a pipeline.    
*Key steps:*    
1. Validate that at least one argument was supplied.    
2. If only `"help"` is given, delegate to `returnHelp`.    
3. Call `parsePipedCommand(args)` (defined elsewhere) to split the command string into individual commands.    
4. For each parsed command:    
   - Resolve full executable path via `prepareArguments`.    
   - Build an `exec.Cmd` with context and arguments.    
   - Log the constructed command.    
5. Execute all built commands in a pipeline using `execPipedCommand(stream, cmds...)`.  
  
---  
  
### 6. `prepareArguments` helper    
*Purpose:* Resolve the full path of the executable and prepend policy directory to the argument list.    
*Key steps:*    
1. Call `resolveExecPath(args[0])`.    
2. Append the resolved path and remaining arguments into a new slice.  
  
---  
  
### 7. `resolveExecPath` helper    
*Purpose:* Find the absolute path of an executable using `exec.LookPath`.    
*Key steps:*    
1. Look up the binary name in the system PATH.    
2. Return the found path or an error.  
  
---  
  
### 8. `returnHelp` helper    
*Purpose:* Produce a help message listing all available commands (similar to Banner).    
*Key steps:*    
1. Read directory entries with `ioutil.ReadDir`.    
2. Build a string containing each command name prefixed by `- `.    
3. Send the resulting byte slice as a single chunk via the gRPC server.  
  
---  
  
# secsh/watch.go  
**Package / Imports**  
  
- **Package name:** `secsh`    
- **Imports used:**  
  - `context`  
  - `os`  
  - `time`  
  - `go.uber.org/zap`  
  
---  
  
### External data, input sources  
  
| Source | Description |  
|--------|-------------|  
| `ctx context.Context` | Execution context that can be cancelled or timed out. |  
| `path string` | Filesystem path to watch for existence. |  
| `interval time.Duration` | Polling interval between checks. |  
| `log *zap.SugaredLogger` | Structured logger used for status messages. |  
  
---  
  
### TODO comments  
  
No explicit TODO markers are present in the current file.  
  
---  
  
## Summary of major code parts  
  
### WatchDir function  
- **Purpose**: Blocks until a directory at the specified path exists or the provided context is cancelled.  
- **Key steps**:  
  1. Create a ticker that fires every `interval` duration (`time.NewTicker(interval)`).  
  2. In an infinite loop, wait for either the context to be done or the ticker channel to fire.  
  3. On each tick, check if the path exists using `os.Stat`. If it does, return `nil`; otherwise log a waiting message and continue looping.  
- **Return value**: Returns any error from the context’s Done channel (typically `ctx.Err()` when cancelled).  
  
---  
  
