# cmd/cli/config/config.go  
# Package `config`  
  
## Imports  
```go  
import (  
    "fmt"  
    "io/ioutil"  
    "os"  
    "path"  
  
    "github.com/jinzhu/configor"  
    "github.com/sonm-io/core/accounts"  
    "github.com/sonm-io/core/insonmnia/auth"  
    "github.com/sonm-io/core/util"  
    "gopkg.in/yaml.v2"  
)  
```  
  
## External data & input sources  
| Source | Description |  
|--------|-------------|  
| `configName` (`cli.yaml`) | Default configuration file name. |  
| `util.GetDefaultConfigDir()` | Provides the default directory for config files if no path is supplied. |  
| YAML marshaling/unmarshaling via `gopkg.in/yaml.v2` | Reads/writes the configuration structure to/from disk. |  
| `accounts.EthConfig` & `auth.ParseAddr` | Types and helper functions used in the configuration struct. |  
  
## TODOs  
No explicit TODO comments were found in this file.  
  
---  
  
# Summary of major code parts  
  
## Constants  
Defines three constants:  
- `OutputModeSimple` – a string constant for simple output mode.  
- `OutputModeJSON` – a string constant for JSON output mode.  
- `configName` – the default config file name (`cli.yaml`).  
  
These are used to set defaults and locate the configuration file.  
  
## `Config` struct  
```go  
type Config struct {  
    Eth        accounts.EthConfig `yaml:"ethereum"`  
    OutFormat  string             `required:"false" default:"" yaml:"output_format"`  
    WorkerAddr string             `yaml:"worker_eth_addr"`  
    NodeAddr   string             `yaml:"node_addr"`  
    path       string  
}  
```  
The struct holds:  
- Ethereum configuration (`Eth`).  
- Output format string.  
- Addresses for worker and node.  
- Internal file path.  
  
Tags indicate how the fields map to YAML keys. The `path` field is internal and not marshaled directly.  
  
## `NewConfig(p ...string) (*Config, error)`  
Creates a new configuration instance:  
1. Calls `getConfigPath` to determine where the config file lives.  
2. Instantiates a `Config` with that path.  
3. If the file does **not** exist, it populates defaults (`fillWithDefaults`) and writes the file immediately.  
4. Otherwise it loads existing YAML data into the struct via `configor.Load`.  
5. Validates the loaded configuration (currently only checks worker address).  
6. Returns the populated config or an error.  
  
## `Validate() error`  
Performs a simple validation: parses the worker address using `auth.ParseAddr`. If parsing fails, returns an error; otherwise nil.  
  
## `Save() error`  
Serializes the current struct to YAML and writes it to disk at the stored path. Uses `ioutil.WriteFile` with mode `0600`.  
  
## Getter helpers  
- `OutputFormat()` – returns the configured output format string.  
- `PassPhrase()` – returns the Ethereum passphrase from the nested config.  
- `KeyStore()` – returns the keystore location from the nested config.  
  
These are convenience accessors for other parts of the package.  
  
## `fillWithDefaults()`  
Sets a default value for `OutFormat` (`OutputModeSimple`). This is called when creating a brand‑new configuration file.  
  
## `getConfigPath(p ...string) (string, error)`  
Helper that resolves the full path to the config file:  
1. If an argument was supplied and non‑empty, it uses that as the base path.  
2. Otherwise it calls `util.GetDefaultConfigDir()` for a default directory.  
3. Checks whether the file already exists; if not, joins the directory with `configName` to produce the final path.  
  
---  
  
This file provides all plumbing needed to read/write a configuration file for the `sonmcli` tool, including defaults, validation, and helper getters.  
  
# cmd/cli/config/config_test.go  
## Package / Component    
**config**  
  
### Imports  
```go  
import (  
	"io/ioutil"  
	"os"  
	"path"  
	"testing"  
  
	"github.com/stretchr/testify/assert"  
	"github.com/stretchr/testify/require"  
)  
```  
  
### External data & input sources    
| Function | Purpose | Input source |  
|----------|---------|--------------|  
| `testConfigDir()` | Creates a temporary directory for tests. | `ioutil.TempDir` |  
| `createTestConfigFile(body string)` | Writes a config file into the temp dir and returns its path. | `os.Mkdir`, `path.Join`, `ioutil.WriteFile` |  
| `deleteTestConfigFile(dir string)` | Removes the created config file after tests. | `path.Join`, `os.Remove` |  
| `getConfigPath("/tmp")` (used in last test) | Resolves a full path to the config file. | `path.Join` |  
  
### TODOs  
No explicit `TODO:` comments were found, but the following areas could be expanded:  
- Add error handling for missing config files.  
- Parameterize default values in tests.  
  
---  
  
## Summary of major code parts  
  
#### 1. Test helpers    
* **`testConfigDir()`** – Generates a temporary directory name using `ioutil.TempDir`.    
* **`createTestConfigFile(body string)`** – Builds the full path to the config file (`configName` is assumed defined elsewhere), writes the supplied body, and returns the directory path.    
* **`deleteTestConfigFile(dir string)`** – Cleans up by removing the written config file.  
  
#### 2. Test cases for `NewConfig`    
All tests follow a similar pattern: create a test config file (or rely on defaults), call `NewConfig`, then assert that the returned configuration object contains expected values.  
  
| Test | What it verifies |  
|------|-------------------|  
| `TestConfigLoad` | Loads a fully populated config and checks all fields (`output_format`, `ethereum.key_store`, `ethereum.pass_phrase`). |  
| `TestConfigDefaults` | Ensures defaults are applied when the file is empty. |  
| `TestConfigNoFile` | Confirms that calling `NewConfig` on an empty directory still yields default values. |  
| `TestConfigCannotRead` | Tests error handling when the config file cannot be read (permissions set to 0200). |  
| `TestGetConfigPath` | Validates that `getConfigPath` correctly appends `/cli.yaml` to a base path. |  
  
#### 3. Assertions & requirements    
All tests use `assert.NoError`, `assert.Equal`, and `require.NoError` from the testify package, ensuring both correctness of returned values and proper error handling.  
  
---  
  
This file provides the core unit‑testing scaffold for the `config` component, covering creation, deletion, loading, defaulting, and path resolution.  
  
