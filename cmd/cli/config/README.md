# Package `config`

## Overview  
The `config` package provides a lightweight configuration loader for the **sonmcli** command‑line tool.  
It reads/writes a YAML file named *cli.yaml* (default) that contains Ethereum and node settings, validates the data, and exposes convenient getters for other parts of the application.

---

## File structure  

```
cmd/
└─ cli/
   └─ config/
      ├─ config.go
      └─ config_test.go
```

---

## Key entities

| Entity | Purpose |
|--------|---------|
| `configName` (constant) | Default file name (`cli.yaml`). |
| `Config` struct | Holds the configuration data. |
| `NewConfig(p ...string)` | Creates or loads a configuration instance. |
| `Validate()` | Checks that the worker address can be parsed. |
| `Save()` | Persists the current config to disk. |
| Getter helpers (`OutputFormat()`, `PassPhrase()`, `KeyStore()`) | Expose nested fields for other packages. |

---

## Environment / runtime variables

* **`configName`** – default file name used by all functions.  
* **`util.GetDefaultConfigDir()`** – provides the base directory when no explicit path is supplied to `NewConfig`.  

No external environment variables or command‑line flags are required; the only runtime argument is an optional path passed to `NewConfig`.

---

## How it works

1. **Path resolution**  
   ```go
   getConfigPath(p ...string) (string, error)
   ```
   * If a non‑empty string is supplied, it is used as the base directory.  
   * Otherwise the default directory from `util.GetDefaultConfigDir()` is taken.  
   * The function appends `configName` (`cli.yaml`) to that directory and returns the full path.

2. **Creating / loading**  
   ```go
   NewConfig(p ...string) (*Config, error)
   ```
   * Calls `getConfigPath`.  
   * If the file does not exist, it calls `fillWithDefaults()` (sets a default output format), writes the file immediately, and then loads it.  
   * Otherwise it simply loads the existing YAML data via `configor.Load`.  
   * The resulting struct is returned for further use.

3. **Validation**  
   ```go
   Validate() error
   ```
   * Currently only checks that the worker address can be parsed by `auth.ParseAddr`.  

4. **Persistence**  
   ```go
   Save() error
   ```
   * Serializes the struct to YAML and writes it back to disk.

5. **Getters** – simple helpers for other packages:
   * `OutputFormat()` – returns the configured output format string.
   * `PassPhrase()` – returns the Ethereum passphrase from the nested config.
   * `KeyStore()` – returns the keystore location from the nested config.

---

## Tests (config_test.go)

* **Helper functions**  
  * `testConfigDir()`, `createTestConfigFile(body string)`, and `deleteTestConfigFile(dir string)` create a temporary directory, write a test file, and clean up afterwards.  

* **Test cases** – verify that:
  * A fully populated config can be loaded (`TestConfigLoad`).  
  * Default values are applied when the file is empty (`TestConfigDefaults`).  
  * `NewConfig` works even if no file exists yet (`TestConfigNoFile`).  
  * Errors are handled correctly when the file cannot be read (`TestConfigCannotRead`).  
  * Path resolution behaves as expected (`TestGetConfigPath`).  

All tests use `assert.NoError`, `require.NoError`, and `assert.Equal` from the testify package.

---

## Edge cases for launching

* **Default launch** – running the binary without arguments will load configuration from `$HOME/.config/cli.yaml` (or whatever `util.GetDefaultConfigDir()` returns).  
* **Explicit path** – passing a directory to `NewConfig`, e.g. `cmd/cli/main.go: config, err := config.NewConfig("/tmp")`, forces the binary to read/write `/tmp/cli.yaml`.  

---

## Summary of logic

The package encapsulates all plumbing needed to read/write a YAML configuration file for *sonmcli*.  
It resolves the file path, provides defaults, validates data, and offers simple getters.  
Tests confirm that each step behaves correctly under various conditions.