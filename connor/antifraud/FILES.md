# connor/antifraud/antifraud.go  
# Package / Component    
**antifraud**  
  
## Imports  
```go  
import (  
	"context"  
	"fmt"  
	"sync"  
	"time"  
  
	"github.com/ethereum/go-ethereum/common"  
	"github.com/prometheus/client_golang/prometheus"  
	"github.com/sonm-io/core/connor/types"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util"  
	"go.uber.org/zap"  
	"go.uber.org/zap/zapcore"  
	"golang.org/x/sync/errgroup"  
	"google.golang.org/grpc"  
)  
```  
  
## External data / input sources  
| Source | Description |  
|--------|-------------|  
| `cfg Config` | Configuration values (quality check interval, blacklist interval, whitelist, etc.) |  
| `nodeConnection *grpc.ClientConn` | GRPC connection to the node that manages deals |  
| `dealFactory types.DealFactory` | Factory for creating deal objects from raw data |  
| `processorFactory ProcessorFactory` | Factory that creates log and pool processors for a deal |  
  
## TODOs  
- `checkDeals`: “async” – currently runs synchronously, could be parallelized.  
- `checkBlacklist`: “save this in DB, load on start” – persistence of blacklist watchers.  
  
---  
  
## Summary of major code parts  
  
### 1. Metrics registration (`init`)  
```go  
var (  
	blacklistedDealCounter = prometheus.NewCounter(prometheus.CounterOpts{  
		Name: "sonm_deals_blacklisted",  
		Help: "Number of deals that were closed with blacklisting",  
	})  
)  
  
func init() {  
	prometheus.MustRegister(blacklistedDealCounter)  
}  
```  
Registers a Prometheus counter for deals that are finished via blacklisting.  
  
### 2. Types  
- **`AntiFraud interface`** – public API: `Run`, `DealOpened`, `TrackTask`, `FinishDeal`.  
- **`dealMeta struct`** – holds a deal, its log processor and pool processor.  
- **`antiFraud struct`** – internal state: mutex, maps of meta data and blacklist watchers, factories, config, node connection, client, logger.  
  
### 3. Constructor (`NewAntiFraud`)  
Creates an `antiFraud` instance, initializing all fields, including a GRPC client for deal management and a named logger.  
  
### 4. Main loop (`Run`)  
Starts two tickers: one for quality checks, another for blacklist checks. It blocks until the context is cancelled, invoking `checkDeals` and `checkBlacklist` on each tick.  
  
### 5. Deal quality check (`checkDeals`)  
Iterates over all registered deals:  
- Skips if no log processor.  
- Retrieves a blacklist watcher for the supplier.  
- Computes task quality from logs and pool reports.  
- Decides whether to close the deal (if quality below threshold) or keep it running.  
- If closing, increments the counter, calls `finishDealWithRetry`, and marks the watcher as failed; otherwise marks success.  
  
### 6. Blacklist check (`checkBlacklist`)  
Placeholder that iterates over all watchers and triggers an unblacklisting attempt. TODO: persist state to DB on startup.  
  
### 7. Task tracking (`TrackTask`)  
Registers processors for a deal’s task:  
- Locks meta map, creates pool and log processors via the factory.  
- Runs both processors concurrently using `errgroup`.  
- Returns when both finish.  
  
### 8. Deal opening handler (`DealOpened`)  
Adds a new deal to the internal map and creates a blacklist watcher if one does not already exist.  
  
### 9. Deal finishing orchestration (`FinishDeal` & helpers)  
- **`FinishDeal`** decides which blacklist type to use (worker or nobody) and calls `finishDealWithRetry`.  
- **`finishDealWithRetry`** repeatedly attempts to finish the deal until success, using a ticker of 10 s.  
- **`finishDeal`** performs the actual RPC call to close the deal, checking status first.  
  
### 10. Helper functions  
- **`lifeTime(deal *sonm.Deal)`** – returns how long a deal has been alive.  
- **`whoToBlacklist`** – chooses blacklist type based on quality metrics and whitelist presence.  
- **`isAddressWhitelisted`** – checks if a supplier address is in the configured whitelist.  
  
---  
  
All major parts are now summarized.  
  
# connor/antifraud/antifraud_test.go  
# Package / Component    
**antifraud**  
  
## Imports  
| Import | Purpose |  
|--------|---------|  
| `testing` | Go testing framework – used for unit test execution. |  
| `github.com/ethereum/go-ethereum/common` | Provides Ethereum address handling (`common.Address`) and helper functions such as `HexToAddress`. |  
| `github.com/stretchr/testify/assert` | Assertion library to validate expected outcomes in tests. |  
  
## External Data / Input Sources  
* **Whitelist addresses** – three hard‑coded Ethereum addresses are supplied via the `Config{}` struct:  
  * `0x38FeB5FE6fb1EECb2e0b990214BC6f0ACa1eCE6A`  
  * `0xeEDC24cdcA34fcDFCeaAd84a7C6bA3301D5D707f`  
  * `0xBAdc46A8ca03bD0D458242E76Bc2810DC5f12908`  
  
## TODOs  
No explicit TODO comments are present in this file.  
  
---  
  
# Summary of Major Code Parts  
  
### Test Setup  
```go  
a := &antiFraud{cfg: Config{  
    Whitelist: []common.Address{  
        common.HexToAddress("..."),  
        ...  
    },  
}}  
```  
* Instantiates an `antiFraud` instance with a configuration that contains the whitelist.    
* The struct `antiFraud` and type `Config` are defined elsewhere in the package; this test focuses on the method `isAddressWhitelisted`.  
  
### Whitelist Verification  
```go  
notInList := a.isAddressWhitelisted(common.HexToAddress("0x2b7D9Af99CE1Bec0dc32aB115B4DA82BF15B501E"))  
assert.False(t, notInList)  
```  
* Calls `isAddressWhitelisted` with an address that is **not** in the whitelist and asserts that the method returns `false`.  
  
```go  
inList := a.isAddressWhitelisted(common.HexToAddress("0xeEDC24cdcA34fcDFCeaAd84a7C6bA3301D5D707f"))  
assert.True(t, inList)  
```  
* Calls the same method with an address that **is** present in the whitelist and asserts that it returns `true`.  
  
The test therefore verifies both negative and positive cases for the whitelist lookup logic.  
  
---  
  
# connor/antifraud/blacklist_watcher.go  
**Package name**    
`antifraud`  
  
---  
  
## Imports  
```go  
import (  
	"context"  
	"time"  
  
	"github.com/ethereum/go-ethereum/common"  
	"github.com/sonm-io/core/proto"  
	"go.uber.org/zap"  
	"google.golang.org/grpc"  
)  
```  
* `context` – standard Go context handling    
* `time` – time utilities for durations and timestamps    
* `github.com/ethereum/go-ethereum/common` – Ethereum address type (`common.Address`)    
* `github.com/sonm-io/core/proto` – SonM protocol client definitions (used to create a blacklist client)    
* `go.uber.org/zap` – structured logger from Uber’s Zap library    
* `google.golang.org/grpc` – gRPC connection for the SonM client  
  
---  
  
## External data / input sources  
| Source | Type | Description |  
|--------|------|-------------|  
| `addr common.Address` | Ethereum address | The wallet to watch |  
| `cc *grpc.ClientConn` | gRPC client connection | Connection used by the SonM blacklist client |  
| `log *zap.Logger` | Logger | Used for debug/info output |  
  
---  
  
## TODOs  
No explicit TODO comments are present in this file.  
  
---  
  
# Summary of major code parts  
  
### 1. Struct definition – `blacklistWatcher`  
```go  
type blacklistWatcher struct {  
	log     *zap.Logger  
	address common.Address  
	client  sonm.BlacklistClient  
  
	nextPeriod    time.Duration  
	unBlacklistAt time.Time  
	lastSuccess   time.Time  
}  
```  
* Holds the logger, watched address, and a SonM client.    
* Tracks the next period of blacklisting (`nextPeriod`), when to remove from blacklist (`unBlacklistAt`) and the last successful check timestamp (`lastSuccess`).  
  
### 2. Constructor – `NewBlacklistWatcher`  
```go  
func NewBlacklistWatcher(addr common.Address, cc *grpc.ClientConn, log *zap.Logger) *blacklistWatcher {  
	return &blacklistWatcher{  
		log:        log.Named("blacklist").With(zap.String("wallet", addr.Hex())),  
		address:    addr,  
		nextPeriod: minStep,  
		client:     sonm.NewBlacklistClient(cc),  
	}  
}  
```  
* Creates a new watcher, names the logger “blacklist”, attaches the wallet hex for easier debugging, sets the initial period to `minStep`, and builds a SonM blacklist client from the gRPC connection.  
  
### 3. Failure handling – `Failure()`  
```go  
func (m *blacklistWatcher) Failure() {  
	m.unBlacklistAt = time.Now().Add(m.nextPeriod)  
	m.lastSuccess = time.Time{}  
  
	m.nextPeriod *= 2  
	if m.nextPeriod > maxStep {  
		m.nextPeriod = maxStep  
	}  
  
	m.log.Debug("failure", zap.Duration("step", m.nextPeriod))  
}  
```  
* Marks the next unblacklist time as now + current period.    
* Resets `lastSuccess`.    
* Doubles the period for the next cycle, capped at `maxStep`.    
* Logs a debug message with the new step duration.  
  
### 4. Success handling – `Success()`  
```go  
func (m *blacklistWatcher) Success() {  
	if m.lastSuccess.IsZero() {  
		m.lastSuccess = time.Now()  
		return  
	}  
  
	d := time.Now().Sub(m.lastSuccess)  
	m.nextPeriod -= d  
	if m.nextPeriod < minStep {  
		m.nextPeriod = minStep  
	}  
	m.lastSuccess = time.Now()  
}  
```  
* If this is the first success, simply records the timestamp.    
* Otherwise calculates how long it took since last success (`d`), reduces the period by that amount (but never below `minStep`), and updates `lastSuccess`.  
  
### 5. Utility – `isBlacklisted()`  
```go  
func (m *blacklistWatcher) isBlacklisted() bool {  
	return time.Now().Before(m.unBlacklistAt)  
}  
```  
* Returns whether the current time is still before the scheduled unblacklist moment.  
  
### 6. Unblacklisting operation – `TryUnblacklist(ctx context.Context)`  
```go  
func (m *blacklistWatcher) TryUnblacklist(ctx context.Context) error {  
	if m.isBlacklisted() || m.unBlacklistAt.IsZero() {  
		return nil  
	}  
  
	m.log.Info("removing from blacklist on market")  
	ctx, cancel := context.WithTimeout(ctx, unBlacklistTimeout)  
	defer cancel()  
	if _, err := m.client.Remove(ctx, sonm.NewEthAddress(m.address)); err != nil {  
		m.log.Warn("cannot remove address from blacklist", zap.Error(err))  
		return err  
	}  
  
	m.unBlacklistAt = time.Time{}  
	return nil  
  
}  
```  
* Checks if the watcher is still blacklisted; if not, exits.    
* Logs an info message, creates a timeout context (`unBlacklistTimeout`), and calls the SonM client’s `Remove` method to delete the address from the blacklist.    
* On success clears `unBlacklistAt`.    
  
---  
  
# connor/antifraud/blacklist_watcher_test.go  
**Package / Component**    
`antifraud`  
  
---  
  
### Imports  
```go  
import (  
	"context"  
	"testing"  
	"time"  
  
	"github.com/ethereum/go-ethereum/common"  
	"github.com/sonm-io/core/proto"  
	"github.com/stretchr/testify/assert"  
	"github.com/stretchr/testify/require"  
	"go.uber.org/zap"  
	"google.golang.org/grpc"  
)  
```  
  
---  
  
### External data / input sources  
| Source | Description |  
|--------|-------------|  
| `common.HexToAddress` | Converts a hex string into an Ethereum address used to initialise the watcher. |  
| `minStep` | A package‑level constant (not shown here) that defines the minimal period for the watcher. |  
| `zap.NewNop()` | Provides a no‑op logger instance for the watcher. |  
  
---  
  
### TODOs  
* The comment at the top explains why a manual mock is used instead of `mockgen`:  
  ```go  
  // avoid using the mockgen because it cannot properly mock stream method  
  // that are also declared in the proto/node.pb.go  
  ```  
  
---  
  
## Summary of major code parts  
  
### 1. Mock client implementation    
```go  
type blacklistClientMock struct{}  
```  
* Provides a lightweight stub for the `blacklistWatcher` client interface.  
* Implements three RPC methods (`List`, `Remove`, `Purge`) that simply return empty responses, enabling unit tests without external dependencies.  
  
### 2. Test helper – `newTestBlacklistWatcher()`    
```go  
func newTestBlacklistWatcher() blacklistWatcher { … }  
```  
* Constructs a ready‑to‑use `blacklistWatcher` instance with:  
  * A hard‑coded Ethereum address (`0x950B346f1028cbf76a6ed721786eBcfb13DAc4Ec`).  
  * The minimal period (`minStep`) and the current time as the last success timestamp.  
  * The mock client defined above, and a no‑op logger.  
  
### 3. Unit test – `TestBlackListWatcher()`    
```go  
func TestBlackListWatcher(t *testing.T) { … }  
```  
* Validates the state transitions of the watcher:  
  1. Initially not blacklisted.  
  2. After calling `Success()`, still not blacklisted (no failure yet).  
  3. After a simulated failure (`Failure()`), becomes blacklisted.  
  4. Another success keeps it blacklisted.  
 5. Simulates an un‑blacklisting event and verifies that the watcher correctly clears its state.  
  
The test uses `assert` and `require` from the testify package to check expectations, and relies on the mock client for RPC calls.  
  
---  
  
All of these pieces together provide a minimal yet functional test harness for the `antifraud` component’s blacklist watching logic.  
  
# connor/antifraud/config.go  
# antifraud package  
  
## Imports    
```go  
import (  
	"fmt"  
	"time"  
  
	"github.com/ethereum/go-ethereum/common"  
)  
```  
* `fmt` – used for error formatting in the validation method.    
* `time` – provides the `Duration` type for interval fields.    
* `github.com/ethereum/go-ethereum/common` – supplies the `Address` type used in the whitelist.  
  
---  
  
## External data / input sources    
| Struct | Field | Type | YAML tag | Notes |  
|--------|-------|------|----------|-------|  
| `ProcessorConfig` | `Format` | string | `yaml:"format"` | format of the processor |  
| | `TrackInterval` | time.Duration | `yaml:"track_interval"` | default 10 s |  
| | `TaskWarmupDelay` | time.Duration | `yaml:"warmup_delay"` | required |  
| | `DecayTime` | float64 | `yaml:"decay_time"` | required |  
| `LogProcessorConfig` | (inline) | ProcessorConfig | `yaml:",inline"` | inherits all fields of `ProcessorConfig` |  
| | `Pattern` | string | `yaml:"pattern"` | required |  
| | `Field` | int | `yaml:"field"` | field index to process |  
| | `Multiplier` | float64 | `yaml:"multiplier"` | required |  
| | `LogDir` | string | `yaml:"log_dir"` | directory for logs |  
| `PoolProcessorConfig` | (inline) | ProcessorConfig | `yaml:",inline"` | inherits all fields of `ProcessorConfig` |  
| | `URL` | string | `yaml:"url"` | URL to pool data source |  
| `Config` | `TaskQuality` | float64 | `yaml:"task_quality"` | required |  
| | `QualityCheckInterval` | time.Duration | `yaml:"quality_check_interval"` | default 15 s |  
| | `BlacklistCheckInterval` | time.Duration | `yaml:"blacklist_check_interval"` | default 5 m |  
| | `ConnectionTimeout` | time.Duration | `yaml:"connection_timeout"` | default 60 s |  
| | `LogProcessorConfig` | LogProcessorConfig | `yaml:"log_processor"` | nested config for log processing |  
| | `PoolProcessorConfig` | PoolProcessorConfig | `yaml:"pool_processor"` | nested config for pool processing |  
| | `Whitelist` | []common.Address | `yaml:"whitelist"` | list of addresses to monitor |  
  
---  
  
## TODOs    
No explicit TODO comments are present in the file.  
  
---  
  
# Summary of major code parts  
  
### 1. Package declaration & imports  
The file declares the `antifraud` package and pulls in standard library packages (`fmt`, `time`) plus an external Ethereum common package for address handling.  
  
### 2. Configuration structs  
* **ProcessorConfig** – base configuration shared by both log and pool processors, containing format, timing, and decay parameters.  
* **LogProcessorConfig** – extends `ProcessorConfig` inline with additional fields specific to log processing (pattern, field index, multiplier, directory).  
* **PoolProcessorConfig** – also extends `ProcessorConfig`, adding a URL for the pool data source.  
* **Config** – top‑level configuration that aggregates quality metrics, timing intervals, nested processor configs, and a whitelist of Ethereum addresses.  
  
### 3. Validation method  
`func (c Config) Validate() error` checks that both embedded `DecayTime` values are positive, returning an informative error if not. This ensures the config is sane before use.  
  
---  
  
All parts together provide a lightweight yet extensible configuration model for an anti‑fraud component, ready to be populated from YAML and validated at runtime.  
  
# connor/antifraud/flags.go  
# Package: `antifraud`  
  
## Imports  
No external imports are used in this file.  
  
## External Data / Input Sources  
None – the package only defines constants, a type and a method.  
  
## TODOs  
No TODO comments were found in the provided code.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. Constants  
```go  
const (  
    AllChecks = iota          // 0  
    SkipBlacklisting           // 1  
)  
```  
* `AllChecks` and `SkipBlacklisting` are defined using Go’s `iota`.    
  * `AllChecks` evaluates to `0`, representing a flag that indicates all checks should be performed.    
  * `SkipBlacklisting` evaluates to `1`, used as a bitmask for the `flags` type.  
  
### 2. Type Definition  
```go  
type flags int  
```  
* A simple alias of `int`. The type is intended to hold bitwise flag values that can be combined with the constants above.  
  
### 3. Method on `flags`  
```go  
func (f flags) SkipBlacklist() bool {  
    return int(f)&SkipBlacklisting == 1  
}  
```  
* Receives a value of type `flags` and returns a boolean indicating whether the `SkipBlacklisting` bit is set.  
* The method performs a bitwise AND between the integer representation of `f` and the constant `SkipBlacklisting`, then checks if the result equals `1`.    
  * This effectively tests whether the second flag (bit 1) is active.  
  
---  
  
The file provides a minimal flag system for an antifraud package, allowing callers to check whether blacklisting should be skipped via the `SkipBlacklist()` method.  
  
# connor/antifraud/flags_test.go  
# Package Overview    
**Package name:** `antifraud`    
  
## Imports  
```go  
import (  
	"testing"  
  
	"github.com/stretchr/testify/assert"  
)  
```  
* `testing` – Go's standard testing package, used for unit tests.    
* `github.com/stretchr/testify/assert` – Testify assertion library to simplify test assertions.  
  
---  
  
## External Data / Input Sources    
The file contains a single test function that exercises the `flags` type and its methods. No external files or data sources are referenced directly; it relies on the `flags` implementation defined elsewhere in the same package.  
  
---  
  
## TODOs    
No explicit TODO comments were found in this file.  
  
---  
  
# Summary of Major Code Parts    
  
## TestFlags Function  
```go  
func TestFlags(t *testing.T) {  
	var f flags = SkipBlacklisting  
	assert.True(t, f.SkipBlacklist())  
  
	f = AllChecks  
	assert.False(t, f.SkipBlacklist())  
}  
```  
* **Purpose** – Verify that the `flags` type correctly interprets two flag constants:    
  * `SkipBlacklisting` should cause `f.SkipBlacklist()` to return `true`.    
  * `AllChecks` should cause it to return `false`.    
* **Behavior** – The test initializes a variable `f` of type `flags` with the constant `SkipBlacklisting`, then asserts that calling its method `SkipBlacklist()` yields `true`. It subsequently reassigns `f` to the constant `AllChecks` and checks that the same method now returns `false`.    
* **Assumptions** – The constants `SkipBlacklisting` and `AllChecks` as well as the method `SkipBlacklist()` are defined elsewhere in the package.    
  
This test ensures that flag handling logic behaves as expected for these two cases, providing a quick regression check when changes to the flags implementation occur.  
  
---  
  
# connor/antifraud/log_processor.go  
**Package name**    
`antifraud`  
  
---  
  
## Imports  
  
| Package | Purpose |  
|---------|---------|  
| `bufio` | Buffered I/O for reading logs |  
| `context` | Context handling for goroutines |  
| `fmt` | String formatting and logging |  
| `io` | Pipe creation between reader/writer |  
| `math` | EWMA calculation |  
| `os` | File operations (history file) |  
| `path` | Path construction for log files |  
| `strconv` | Parsing numeric values from strings |  
| `strings` | String manipulation (pattern matching, field extraction) |  
| `time` | Timing and tickers |  
  
Third‑party packages:  
  
* `github.com/docker/docker/pkg/stdcopy` – copy logs between client and writer  
* `github.com/rcrowley/go-metrics` – EWMA for hashrate tracking  
* `github.com/sonm-io/core/connor/types` – Deal type definition  
* `github.com/sonm-io/core/proto` – Protobuf definitions (used in request)  
* `github.com/sonm-io/core/util` – ImmediateTicker helper  
* `go.uber.org/atomic` – Atomic float64 for hashrate  
* `go.uber.org/zap` – Structured logger  
* `google.golang.org/grpc` – GRPC client connection  
* `google.golang.org/grpc/metadata` – Metadata handling in context  
  
---  
  
## External data / input sources  
  
| Variable | Source |  
|----------|--------|  
| `cfg *LogProcessorConfig` | Configuration passed to constructor (contains LogDir, DecayTime, TaskWarmupDelay, TrackInterval, Pattern, Field, Multiplier) |  
| `log *zap.Logger` | Logger instance for the task |  
| `conn *grpc.ClientConn` | GRPC connection used by `sonm.NewWorkerClient` |  
| `deal *types.Deal` | Deal information (ID, benchmark value) |  
| `taskID string` | Identifier of the current task |  
  
The processor also creates a GRPC client (`sonm.WorkerClient`) and uses it to request logs via `sonm.TaskLogsRequest`.  
  
---  
  
## TODO list  
  
No explicit `TODO:` comments were found in this file.  
  
---  
  
## Summary of major code parts  
  
### 1. Constructor – `newLogProcessor`  
  
Creates a new `logProcessor` instance, wiring together the logger, configuration, deal, task ID and GRPC client.    
Initializes an EWMA with a decay factor derived from `cfg.DecayTime`, records the start time, and sets an atomic float64 for hashrate based on the deal’s benchmark value.  
  
### 2. Quality & identification helpers  
  
* `TaskQuality()` – Returns whether the warm‑up period is finished and the current hashrate relative to the desired benchmark.  
* `TaskID()` – Simple accessor returning the task ID string.  
  
### 3. Main run loop – `Run`  
  
* Updates the EWMA with the latest hashrate value every 5 s, then ticks it each second.  
* Starts a goroutine that fetches logs (`fetchLogs`) and waits for a warm‑up delay before entering an infinite ticker loop that keeps updating the EWMA and ticking it.  
  
### 4. History file handling – `maybeOpenHistoryFile`  
  
Creates (or reopens) a log file in the configured directory, naming it with the deal ID and task ID. The file is stored in `m.historyFile` for later writes.  
  
### 5. Log line persistence – `maybeSaveLogLine`  
  
If a history file exists, prepends a timestamp to each incoming log line and appends it to the file.  
  
### 6. Log fetching – `fetchLogs`  
  
* Builds a `sonm.TaskLogsRequest` with type `BOTH`, the task ID, and the deal ID.  
* Opens the history file (if not already open) and starts a retry ticker based on `cfg.TrackInterval`.  
* In each tick: sends the request via the GRPC client, creates a pipe between a reader and writer, launches a goroutine that parses the incoming stream (`logParser`), then copies data from the writer to the writer using `stdcopy.StdCopy`. Errors are logged but not returned.  
  
### 7. Log parsing – `logParser`  
  
* Reads lines from the provided `io.Reader` with a buffered scanner.  
* For each line: writes it to history file, checks if the configured pattern is present, splits into fields, extracts the field at index `cfg.Field`, parses it as a float64 and stores it in the atomic hashrate variable (multiplied by `cfg.Multiplier`).    
  Errors during parsing are logged.  
  
---  
  
All of these parts together provide a self‑contained log processor that fetches logs from a GRPC worker, persists them to disk, and tracks hashrate using an EWMA.  
  
# connor/antifraud/log_processor_test.go  
**Package name:** `antifraud`    
  
**Imports collected**  
  
```go  
import (  
	"context"  
	"fmt"  
	"strings"  
	"testing"  
  
	"github.com/stretchr/testify/assert"  
	"go.uber.org/atomic"  
	"go.uber.org/zap"  
)  
```  
  
---  
  
## External data / input sources  
  
| Source | Description |  
|--------|-------------|  
| `strings.NewReader` | Used in all test functions to provide a log line or a series of lines for the parser. |  
| `mklog(n int)` | Generates a string containing *n* identical log entries, each formatted as “ETH - Total Speed: …”. This is used only in the cancellation‑context test. |  
  
---  
  
## TODO list  
  
No explicit `TODO` comments are present in this file.  
  
---  
  
## Summary of major code parts  
  
### 1. `mklog`  
Creates a multiline string that simulates *n* log entries for testing purposes.  
```go  
func mklog(n int) string {  
	var s string  
	for i := 0; i < n; i++ {  
		s += fmt.Sprintf("ETH - Total Speed: %d.000 Mh/s, Total Shares: 127, Rejected: 0, Time: 00:02\n", i)  
	}  
	return s  
}  
```  
*Purpose:* Provide a predictable input for the parser when testing context cancellation.*  
  
### 2. `newTestProcessor`  
Initializes a `logProcessor` instance with sane defaults.  
```go  
func newTestProcessor() *logProcessor {  
	return &logProcessor{  
		log:      zap.NewNop(),  
		hashrate: atomic.NewFloat64(0),  
		cfg: &LogProcessorConfig{  
			Pattern:    "Total Speed:",  
			Field:      4,  
			Multiplier: 1000000,  
		},  
	}  
}  
```  
*Purpose:* Create a processor ready to parse log lines; the config indicates which field contains the speed value.*  
  
### 3. `TestClaymoreLogParser`  
Unit test that verifies parsing of a single Claymore‑style log line.  
```go  
func TestClaymoreLogParser(t *testing.T) {  
	rd := strings.NewReader(`ETH - Total Speed: 100.000 Mh/s, Total Shares: 127, Rejected: 0, Time: 00:02`)  
	p := newTestProcessor()  
  
	p.logParser(context.Background(), rd)  
	assert.Equal(t, float64(100e6), p.hashrate.Load(), "new value should be parsed and set")  
}  
```  
*Checks:* After parsing, the processor’s `hashrate` field must contain `100 000 000`.*  
  
### 4. `TestClaymoreLogParser_InvalidLine`  
Ensures that an unrelated line does not alter the existing hashrate.  
```go  
func TestClaymoreLogParser_InvalidLine(t *testing.T) {  
	rd := strings.NewReader(`Oops! Claymore failed`)  
	p := newTestProcessor()  
	p.hashrate = atomic.NewFloat64(100500)  
  
	p.logParser(context.Background(), rd)  
	assert.Equal(t, float64(100500), p.hashrate.Load(), "previous value should be kept")  
}  
```  
*Checks:* The processor keeps the previous hashrate when no matching pattern is found.*  
  
### 5. `TestClaymoreLogParser_ShortLine`  
Tests parsing of a minimal line that contains only the pattern.  
```go  
func TestClaymoreLogParser_ShortLine(t *testing.T) {  
	rd := strings.NewReader(`Total Speed:`)  
	p := newTestProcessor()  
  
	p.logParser(context.Background(), rd)  
	assert.Equal(t, float64(0), p.hashrate.Load(), "previous value should be kept")  
}  
```  
*Checks:* The hashrate remains `0` when the line is too short to provide a numeric value.*  
  
### 6. `TestClaymoreLogParser_ContextCancel`  
Verifies that the parser works correctly even if the context is cancelled before parsing.  
```go  
func TestClaymoreLogParser_ContextCancel(t *testing.T) {  
	ctx, cancel := context.WithCancel(context.Background())  
  
	rd := strings.NewReader(mklog(1000))  
	p := &logProcessor{log: zap.NewNop(), hashrate: atomic.NewFloat64(1.2345)}  
	cancel()  
  
	p.logParser(ctx, rd)  
	assert.Equal(t, float64(1.2345), p.hashrate.Load(), "previous value should be kept")  
}  
```  
*Checks:* After cancellation and parsing a large log string, the processor’s hashrate remains unchanged (the test focuses on context handling).    
  
---  
  
**<end_of_output>**  
  
# connor/antifraud/pool_processor.go  
# Package: `antifraud`  
  
## Imports  
```go  
import (  
	"context"  
	"encoding/json"  
	"fmt"  
	"math"  
	"time"  
  
	"github.com/ethereum/go-ethereum/metrics"  
	"github.com/sonm-io/core/connor/price"  
	"github.com/sonm-io/core/connor/types"  
	"github.com/sonm-io/core/util"  
	"go.uber.org/atomic"  
	"go.uber.org/zap"  
	"gopkg.in/oleiade/lane.v1"  
)  
```  
The file pulls in standard packages for context handling, JSON encoding, formatting, math and time utilities; plus third‑party libraries for metrics (EWMA), price fetching, deal types, atomic values, structured logging, and a lane queue.  
  
## External Data / Input Sources  
| Source | Purpose |  
|--------|---------|  
| `price.FetchURLWithRetry(url)` | Retrieves raw JSON from the pool’s API endpoint. |  
| `types.Deal` | Deal configuration and benchmark value used to initialise hashrate metrics. |  
| `metrics.EWMA` | Exponential weighted moving average for smoothing hashrate readings. |  
| `lane.Queue` | Circular buffer that stores recent hashrate samples (60‑slot capped deque). |  
  
## TODOs  
No explicit `TODO:` comments are present in the current file.  
  
---  
  
# Core Structures  
  
### `commonPoolProcessor`  
Represents a worker that polls a mining pool, keeps track of its own hashrate and updates an EWMA.    
Fields:  
- `cfg` – configuration pointer.  
- `log` – zap logger with contextual fields.  
- `taskID`, `workerID` – identifiers for the deal/task.  
- `deal` – reference to the deal being processed.  
- Timing & metrics: `startTime`, `currentHashrate`, `hashrateEWMA`, `hashrateQueue`.  
- `update` – function pointer that fetches pool data (either dwarf or uley).  
  
### `dwarfPoolWorker` / `dwarfPoolResponse`  
JSON‑serialisable structs for the *dwarfpool* API.    
The response contains a map of workers keyed by worker ID.  
  
### `uleyPoolWorker` / `uleyPoolResponse`  
Analogous structs for the *uleypool* API.  
  
---  
  
# Constructors  
  
#### `newDwarfPoolProcessor`  
Creates a `commonPoolProcessor` configured to poll the dwarfpool endpoint.    
- Builds a logger named `"dwarfpool"`.  
- Sets `workerID` as `"c<deal_id>"`.  
- Initializes EWMA with decay factor `1 - exp(-5/DecayTime)`.  
- Assigns `update` to `dwarfPoolUpdateFunc`.  
  
#### `newUleyPoolProcessor`  
Same logic but for the uleypool endpoint.    
- Logger named `"uleypool"`, worker ID prefixed with `"u"`.  
  
---  
  
# Processor Run Loop  
  
The method `Run(ctx)` drives continuous polling:  
1. Updates EWMA and ticks it every second.  
2. Waits an initial warm‑up delay (`TaskWarmupDelay`) before starting the loop.  
3. Uses three tickers:    
   - `ewmaTick` (5 s) – updates EWMA value.    
   - `ewmaUpdate` (1 s) – ticks EWMA.    
   - `track` (interval from config) – triggers a pool data fetch via the stored `update` function.  
4. On each track tick, it:  
   - Calls the update function to get a new hashrate value.  
   - Stores that value in `currentHashrate`.  
   - Pushes it into the circular queue with `updateHashRateQueue`.  
  
---  
  
# Helper Methods  
  
- `TaskID()` – returns the task identifier.  
- `TaskQuality()` – computes whether the pool is warm‑up and returns a quality ratio (actual/desired hashrate).  
- `updateHashRateQueue(v)` – appends or rotates the queue to keep it full.  
- `nonZeroHashrate()` – checks if at least five samples exist in the queue.  
  
---  
  
# Update Functions  
  
#### `dwarfPoolUpdateFunc`  
Fetches JSON from a dwarfpool URL, unmarshals into `dwarfPoolResponse`, extracts the worker by ID, and returns its calculated hashrate (scaled to µW).  
  
#### `uleyPoolUpdateFunc`  
Same pattern for uleypool: fetches, unmarshals into `uleyPoolResponse`, pulls the worker data, and returns its effective hashrate.  
  
---  
  
# connor/antifraud/pool_processor_test.go  
# Package / Component    
**Package name:** `antifraud`    
  
## Imports  
```go  
import (  
	"testing"  
  
	"github.com/stretchr/testify/assert"  
	"gopkg.in/oleiade/lane.v1"  
)  
```  
* `testing` – standard Go testing package.    
* `github.com/stretchr/testify/assert` – assertion helpers for unit tests.    
* `gopkg.in/oleiade/lane.v1` – provides the `lane.Queue` type and a capped deque implementation used by the processor.  
  
## External Data / Input Sources  
The test relies on:  
* A freshly created `commonPoolProcessor` instance that contains a `hashrateQueue`.    
* The queue is initialized with a capped deque of capacity 60 (`lane.NewCappedDeque(60)`).  
  
No external files or configuration are referenced; all data comes from the in‑memory queue.  
  
## TODOs  
There are no explicit `TODO:` comments in this file, but the test name suggests that further tests may be added later.  
  
---  
  
# Summary of Major Code Parts  
  
### 1. Test Setup    
```go  
w := &commonPoolProcessor{  
	hashrateQueue: &lane.Queue{Deque: lane.NewCappedDeque(60)},  
}  
```  
* Instantiates a `commonPoolProcessor` with an empty queue that can hold up to 60 elements.  
  
### 2. Empty Queue Check    
```go  
assert.True(t, w.nonZeroHashrate())  
```  
* Verifies that calling `nonZeroHashrate()` on the freshly created processor returns `true`.    
  This indicates that the method correctly interprets an empty queue as having a non‑zero hashrate (likely because it checks for at least one element).  
  
### 3. Adding Non‑Zero Items    
```go  
for i := 0; i <= 5; i++ {  
	w.updateHashRateQueue(float64(i))  
}  
assert.True(t, w.nonZeroHashrate())  
```  
* Adds six values (`0` through `5`) to the queue via `updateHashRateQueue`.    
* After these updates, another assertion confirms that the processor still reports a non‑zero hashrate.  
  
### 4. Adding Zero Items    
```go  
for i := 0; i < 5; i++ {  
	w.updateHashRateQueue(0)  
}  
assert.False(t, w.nonZeroHashrate())  
```  
* Adds five zero values to the queue and checks that `nonZeroHashrate()` now returns `false`.    
  This tests the processor’s ability to detect a drop in hashrate after consecutive zero updates.  
  
---  
  
The test covers three scenarios for the `commonPoolProcessor`’s hashrate logic: an empty queue, a queue with recent non‑zero values, and a queue that receives zeros. It ensures that the `nonZeroHashrate()` method behaves as expected across these states.  
  
# connor/antifraud/processor.go  
**Package & Imports**    
- **Package name:** `antifraud`    
- **Imports:**  
  - `context`  
  - `github.com/sonm-io/core/connor/types`  
  - `go.uber.org/zap`  
  - `google.golang.org/grpc`  
  
---  
  
### External Data Sources  
| Source | Description |  
|--------|-------------|  
| `Config` | Configuration struct (defined elsewhere) that holds format settings for log and pool processors. |  
| `types.Deal` | Deal data structure from the connor/types package, passed to all processor constructors. |  
| `zap.Logger` | Logger instance used by processors. |  
| `grpc.ClientConn` | GRPC client connection passed as an option to processors. |  
  
---  
  
### TODOs  
No explicit `TODO:` comments are present in this file.  
  
---  
  
## Summary of Major Code Parts  
  
#### Constants  
Defines string constants that identify processor formats:  
- `LogFormatCommon`  
- `PoolFormatDwarf`  
- `PoolFormatUley`  
- `ProcessorFormatDisabled`  
  
These constants drive the selection logic inside `NewProcessorFactory`.  
  
#### Processor Interface  
```go  
type Processor interface {  
    Run(ctx context.Context) error  
    TaskID() string  
    TaskQuality() (accurate bool, quality float64)  
}  
```  
Represents a generic processor that can run asynchronously, expose its task ID and report a quality metric.  
  
#### disabledProcessor Implementation  
A minimal stub implementation of `Processor` used when no real logic is required.    
- Stores a `taskID`.  
- Methods return the stored ID, a fixed quality value, and wait for context completion.  
  
#### ProcessorFactory Interface  
```go  
type ProcessorFactory interface {  
    LogProcessor(deal *types.Deal, taskID string, opts ...Option) Processor  
    PoolProcessor(deal *types.Deal, taskID string, opts ...Option) Processor  
}  
```  
Provides factory methods to create log‑ and pool‑specific processors.  
  
#### NewProcessorFactory Function  
Creates a concrete `processorFactory` based on the supplied configuration:  
1. Builds two builder functions (`pool`, `log`) that close over the config format values.  
2. Uses helper `makeOpts` to convert variadic `Option`s into a single `*processorOpts`.  
3. Returns a `processorFactory` instance with those builders wired.  
  
#### makeOpts Helper  
```go  
func makeOpts(opts ...Option) *processorOpts {  
    o := &processorOpts{}  
    for _, opt := range opts { opt(o) }  
    return o  
}  
```  
Aggregates optional configuration functions into one options struct.  
  
#### processorOpts Struct & Option Type  
- `processorOpts` holds a logger and GRPC client connection.  
- `Option` is a function that mutates a `*processorOpts`.  
  
#### WithLogger / WithClientConn Functions  
Convenience constructors for the two `Option` types:  
```go  
func WithLogger(log *zap.Logger) Option { ... }  
func WithClientConn(cc *grpc.ClientConn) Option { ... }  
```  
  
#### builderFunc Type  
A function signature used by the factory to build processors:  
```go  
type builderFunc func(deal *types.Deal, taskID string, opts ...Option) Processor  
```  
  
#### processorFactory Struct & Methods  
Holds the two builder functions and implements `ProcessorFactory`:  
- `pool` and `log` fields store the respective builders.  
- `LogProcessor` and `PoolProcessor` simply invoke those stored functions.  
  
---  
  
