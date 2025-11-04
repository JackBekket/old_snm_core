# insonmnia/version/version.go  
# Package: `version`  
  
## Imports  
```go  
import (  
	"context"  
	"encoding/json"  
	"fmt"  
	"io"  
	"io/ioutil"  
	"net/http"  
	"os"  
	"path/filepath"  
	"strings"  
	"time"  
  
	"github.com/blang/semver"  
	"github.com/mitchellh/go-homedir"  
	"go.uber.org/zap"  
	"gopkg.in/yaml.v2"  
)  
```  
  
## Constants & Global Data  
| Constant | Description |  
|----------|-------------|  
| `versionFilePath` | Path to the local YAML file that stores the current version (`~/.sonm/version.yaml`). |  
| `versionURI` | GitHub API endpoint used by the default `VersionFetcher`. |  
  
```go  
const (  
	versionFilePath = "~/.sonm/version.yaml"  
	versionURI      = "https://api.github.com/repos/sonm-io/core/git/refs/tags"  
)  
```  
  
## Global Variable  
* `Version string` – holds the current platform version that is shared across components.  
  
## Interfaces & Core Types  
  
| Type | Purpose |  
|------|---------|  
| `Observer` | Callback interface for reporting validation results. |  
| `File` | Abstraction over a file handle (used mainly for testing). |  
| `VersionFetcher` | Strategy to fetch the latest version from an external source. |  
  
### Observer  
```go  
type Observer interface {  
	OnError(err error)  
	OnDeprecatedVersion(version, latestVersion semver.Version)  
	OnBleedingEdgeVersion(version, latestVersion semver.Version)  
}  
```  
  
### LogObserver (implementation of `Observer`)  
```go  
type LogObserver struct { log *zap.SugaredLogger }  
  
func NewLogObserver(log *zap.SugaredLogger) *LogObserver  
func (m *LogObserver) OnError(err error)  
func (m *LogObserver) OnDeprecatedVersion(version, latestVersion semver.Version)  
func (m *LogObserver) OnBleedingEdgeVersion(version, latestVersion semver.Version)  
```  
  
### File & FS abstractions  
```go  
type File interface {  
	Read(p []byte) (int, error)  
	Write(p []byte) (int, error)  
	Close() error  
}  
  
type FS interface {  
	Open() (File, error)  
	Create() (File, error)  
}  
```  
  
## VersionFetcher Implementation  
  
* `gitTag` – simple struct for unmarshalling GitHub tag JSON.    
* `versionFetcher` – concrete implementation of `VersionFetcher`.    
  
```go  
type gitTag struct { Ref string }  
  
type versionFetcher struct { url string }  
func newVersionFetcher(url string) *versionFetcher  
func (m *versionFetcher) Update(ctx context.Context) (semver.Version, error)  
```  
  
The fetcher performs an HTTP GET to the GitHub API, decodes JSON into a slice of `gitTag`, extracts the last tag, and returns it as a `semver.Version`.  
  
## YAML Marshal/Unmarshal  
  
* `versionData` – internal representation used by the manager.    
* `versionRawData` – raw form for YAML marshaling.  
  
```go  
type versionData struct { Version semver.Version; Updated time.Time }  
type versionRawData struct { Version string; Updated string }  
  
func (m *versionData) MarshalYAML() (interface{}, error)  
func (m *versionData) UnmarshalYAML(decoder func(interface{}) error) error  
```  
  
`MarshalYAML` converts the structured data into a raw map that can be marshaled by `yaml.Marshal`.    
`UnmarshalYAML` performs the reverse operation, parsing YAML back into a `versionData`.  
  
## FS Implementation (`fs`)  
  
```go  
type fs struct { path string }  
func (m *fs) Open() (File, error)  
func (m *fs) Create() (File, error)  
```  
  
The implementation uses standard library functions to open or create the version file.  
  
## VersionManager  
  
### Constructor & Options  
```go  
type options struct {  
	Version        semver.Version  
	Clock          func() time.Time  
	ExpireDuration time.Duration  
	VersionFetcher VersionFetcher  
}  
  
func newOptions() *options  
func WithClock(clock func() time.Time) Option  
func WithVersionFetcher(versionFetcher VersionFetcher) Option  
func WithVersion(version semver.Version) Option  
  
func NewVersionManager(fs FS, options ...Option) *VersionManager  
```  
  
`NewVersionManager` accepts an `FS` implementation and optional configuration functions.    
The default clock is `time.Now`, the default fetcher uses `newVersionFetcher(versionURI)`.  
  
### Core Methods  
| Method | Purpose |  
|--------|---------|  
| `ValidateCurrentVersion(ctx context.Context) error` | Checks if the local version matches the latest fetched one; reports via an `Observer`. |  
| `latestVersion(ctx context.Context) (semver.Version, error)` | Reads current data, updates file if expired, writes YAML. |  
| `versionData() (*versionData, error)` | Helper that opens the file and unmarshals its content into a `*versionData`. |  
| `isExpired(time time.Time) bool` | Determines whether the stored version is older than the configured expiration duration. |  
  
```go  
func (m *VersionManager) ValidateCurrentVersion(ctx context.Context) error  
func (m *VersionManager) latestVersion(ctx context.Context) (semver.Version, error)  
func (m *VersionManager) versionData() (*versionData, error)  
func (m *VersionManager) isExpired(time time.Time) bool  
```  
  
## Main Validation Flow (`ValidateVersion`)  
  
```go  
func ValidateVersion(ctx context.Context, observer Observer) {  
	path, err := homedir.Expand(versionFilePath)  
	...  
	manager := NewVersionManager(&fs{path: path})  
	if err := manager.ValidateCurrentVersion(ctx); err != nil { ... }  
}  
```  
  
* Expands the home directory to get an absolute file path.    
* Creates a `VersionManager` with that path.    
* Calls `ValidateCurrentVersion`; any errors are reported through the supplied `Observer`.  
  
## TODOs  
No explicit TODO comments were found in this file, but future improvements could include:  
- Adding unit tests for each method.  
- Enhancing error handling (e.g., retry logic on HTTP failures).  
- Supporting additional version sources beyond GitHub.  
  
---  
  
# insonmnia/version/version_test.go  
## Package name    
`version`  
  
## Imports    
```go  
import (  
	"context"  
	"io"  
	"os"  
	"testing"  
	"time"  
  
	"github.com/blang/semver"  
	"github.com/golang/mock/gomock"  
	"github.com/pkg/errors"  
	"github.com/stretchr/testify/require"  
)  
```  
The package pulls in the standard library packages for context handling, I/O, OS interaction, testing and time parsing.    
External dependencies are:  
* `github.com/blang/semver` – semantic‑version parsing & manipulation  
* `github.com/golang/mock/gomock` – mock objects used in tests  
* `github.com/pkg/errors` – error type wrapper  
* `github.com/stretchr/testify/require` – assertions for test expectations  
  
## External data / input sources    
All unit tests feed a *mock file* with a JSON‑like string that represents the current version state.  The content strings are:  
  
| Test | Content |  
|------|---------|  
| `TestVersionManagerErrFileFailedToOpen` | none (file open fails) |  
| `TestVersionManagerErrFileRead` | none (read error) |  
| `TestVersionManagerErrFileInvalidFormat` | `}{` |  
| `TestVersionManagerErrFileMissingVersion` | `{"updated": "2018-11-27 09:55:21+00:00"}` |  
| `TestVersionManagerErrFileMissingUpdated` | `{"version": "v0.4.16"}` |  
| `TestVersionManagerErrFileInvalidVersion` | `{"version": "vvv", "updated": "2018-11-27 09:55:21+00:00"}` |  
| `TestVersionManagerErrFileInvalidUpdated` | `{"version": "v0.4.16", "updated": "2018-11-27"}` |  
| `TestVersionManagerErrUpdateWhenExpired` | `{"version": "v0.4.16", "updated": "2018-11-20T00:00:00+00:00"}` |  
| `TestVersionManagerErrUpdateWhenExpiredThenWhenCreate` | same as above |  
| `TestVersionManagerErrUpdateWhenExpiredThenWhenSave` | same as above, plus expected output string |  
| `TestVersionManagerErrVersionMismatchAfterUpdate` | same as above |  
| `TestVersionManagerErrVersionMismatchWhenUpdateNotRequired` | `{"version": "v0.4.16", "updated": "2018-11-22T00:00:00+00:00"}` |  
| `TestVersionManagerErrVersionMismatchWhenVersionFileNotExists` | none (file missing) |  
| `TestVersionManager` | none (file missing, create & write) |  
  
The tests also use a mock clock function that returns a fixed time parsed from RFC3339 format.  
  
## TODO comments    
No explicit `TODO:` markers are present in the file; all logic is exercised by the test suite.  
  
---  
  
## Summary of major code parts    
  
### 1. Package initialization    
```go  
func init() {  
	Version = "v0.4.0-f6461c2a"  
}  
```  
Sets a default version string for the package.  This value is used as a baseline in tests that create or update the version file.  
  
### 2. Test helpers & mocks    
All tests instantiate:  
* `NewMockFS` – a mock filesystem interface  
* `NewMockFile` – a mock file object  
* `NewMockVersionFetcher` – a mock fetcher for semantic versions  
  
The tests configure expectations on these mocks (e.g., `Open`, `Read`, `Write`, `Close`) to simulate various I/O scenarios.  
  
### 3. Validation of current version    
Each test calls:  
```go  
m := NewVersionManager(fs, WithClock(clock), WithVersionFetcher(versionFetcher))  
err := m.ValidateCurrentVersion(context.Background())  
```  
The tests then assert that the returned error is of the expected type (`OSError`, `CodecError`, or `VersionMismatchError`) and that it satisfies the test conditions.  
  
### 4. Error scenarios    
* **File open failure** – verifies that an OS permission error propagates correctly.  
* **Read errors** – ensures a read‑time error is wrapped as `OSError`.  
* **Invalid JSON format** – checks parsing of malformed content (`}{`).  
* **Missing fields** – tests handling when either the `"version"` or `"updated"` key is absent.  
* **Update logic** – several tests cover the case where the stored version is older than the fetched one, requiring an update and a subsequent write.  They also test the create‑file path when the file does not exist.  
  
### 5. Success scenario    
The final test (`TestVersionManager`) covers the happy path: the file does not exist, it is created, updated with a new semantic version, and written without error.  
  
---  
  
All tests collectively validate that `ValidateCurrentVersion` correctly reads, parses, compares, updates, and writes the version information while handling all edge cases.  The test suite also implicitly documents the expected behavior of the `VersionManager` type.  
  
