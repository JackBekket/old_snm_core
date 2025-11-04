# task_config

## What the package does  
`task_config` is a small helper library that reads a YAML‑encoded *TaskSpec* protobuf from disk, validates it, and exposes the resulting Go value to callers.  The public API consists of two functions:

| Function | Purpose |
|----------|---------|
| `LoadConfig(path string) (*sonm.TaskSpec, error)` | Reads a file at *path*, decodes it into a `TaskSpec` message, validates that message and returns the pointer. |
| `LoadFromFile(path string, into interface{}) error` | Generic helper used by `config.go`.  It reads raw bytes from *path*, feeds them to a YAML decoder, optionally calls a `Validate()` method on the target value and returns any error. |

The package is intended to be used by other parts of the project that need to load task specifications (e.g., a CLI tool or a build system).  

---

## Environment variables, flags & command‑line arguments  
| Variable / Flag | Default / Example | Purpose |
|------------------|-------------------|---------|
| `TASK_CONFIG_PATH` | – | Path to the YAML file that should be loaded by `LoadConfig`.  The tests use a constant `testCfgPath`, so callers can set this variable or pass an explicit path. |
| `-config` (flag) | – | Optional command‑line flag that could be wired into a CLI tool to override the default config path. |

---

## Edge cases for launching  
* **CLI main** – If a binary is built from `cmd/cli/task_config`, it can be invoked as:  

  ```bash
  go run ./cmd/cli/task_config -config=/path/to/spec.yaml
  ```

  The program would call `LoadConfig(os.Getenv("TASK_CONFIG_PATH"))` (or the flag value) and then use the returned `TaskSpec`.  
* **Test harness** – The test suite creates a temporary file (`test.yaml`) via `createTestConfigFile`, runs `LoadConfig(testCfgPath)` and cleans up with `deleteTestConfigFile()`.  This demonstrates that the loader works both for real files and in unit tests.  

---

## File structure (project package tree)  

```
cmd/
└─ cli/
   ├─ task_config/
   │  ├─ config.go
   │  ├─ config_test.go
   │  └─ load_order.go
```

* `config.go` – Public API for loading a single `TaskSpec`.  
* `config_test.go` – Unit tests exercising the loader and helper functions.  
* `load_order.go` – Generic YAML‑to‑Go loader used by `config.go`.

---

## Relations between code entities  

1. **`LoadConfig`** (in *config.go*) creates a new `sonm.TaskSpec`, then calls `config.LoadWith(cfg, path, config.SnakeToLower)` from the util package.  The helper in *load_order.go* is not directly referenced here; it can be used by other modules that need to load arbitrary structs.  
2. **`LoadFromFile`** (in *load_order.go*) is a thin wrapper around `ioutil.ReadFile`, a YAML decoder, and an optional validation call.  It is generic enough to be reused elsewhere in the project.  
3. The tests in *config_test.go* exercise both functions: they create a test file, invoke `LoadConfig`, then assert that all fields of the returned message are populated correctly.

---

## Summary of logic  

- **Loading** – `LoadFromFile` reads raw bytes from disk and decodes them into any Go value.  It uses strict mode so that missing keys cause an error.  
- **Validation** – After decoding, if the target implements a `Validate() error` method (as `TaskSpec` does), it is called automatically.  
- **Convenience wrapper** – `LoadConfig` simply creates a new `TaskSpec`, delegates to the generic loader, and returns the pointer for callers.

The package therefore provides a clean, testable way to read task specifications from YAML files into protobuf messages, with optional validation and environment‑driven configuration.  

---

## Edge cases of launching (if it is a CLI/command)  

| Scenario | How to launch | What to expect |
|----------|----------------|----------------|
| **Default path** | `go run ./cmd/cli/task_config -config=spec.yaml` | The program will read *spec.yaml* and print or use the resulting `TaskSpec`. |
| **Environment override** | `TASK_CONFIG_PATH=spec.yaml go run ./cmd/cli/task_config` | Uses the env var instead of a flag. |
| **Missing file** | `go run ./cmd/cli/task_config -config=missing.yaml` | The loader will return an error; tests confirm this case (`TestTaskConfigNotExists`). |

---