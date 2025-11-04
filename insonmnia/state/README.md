# state  

The **state** package implements a small persistence layer over BoltDB that stores the runtime
configuration of an *insonmnia* worker.  
It keeps all data in a single JSON blob and exposes typed getters/setters for the most
important fields, while also providing a thin “keyed” wrapper so callers can work with a
specific key/value pair.

---

## Short summary of the file contents  

| Section | What it does |
|---------|---------------|
| **Imports** | Pulls in `context`, `encoding/json`, `sync` and the BoltDB store from Docker’s libkv.  It also uses the local `hardware` package and a Zap‑based logger. |
| **External data / input sources** | The struct `StorageConfig` (not shown but referenced) supplies two values that are passed to `makeStore()` – a BoltDB file path (`Endpoint`) and a bucket name (`Bucket`).  These drive the creation of the underlying `store.Store`. |
| **Core types** | * `stateJSON` – the in‑memory representation of the persisted data.  It contains: <br>• `Benchmarks map[string]bool` – which benchmarks have succeeded.<br>• `Hardware *hardware.Hardware` – a pointer to the hardware description.<br>• `HwHash string` – a hash that can be used for change detection. |
| **Storage struct** | Holds a mutex, a context, the BoltDB store and a pointer to the JSON data.  It offers: <br>• `makeStore()` – creates the BoltDB store from the config.<br>• `newEmptyState()` – builds an empty in‑memory state.<br>• `NewState()` – public constructor that returns a ready‑to‑use `*Storage`. |
| **Persistence helpers** | `dump()` writes the current JSON blob to the store; `loadInitial()` reads it back on startup.  The following typed accessors/mutators are provided: <br>`PassedBenchmarks()`, `SetPassedBenchmarks()`<br>`HardwareHash()`, `SetHardwareHash()`<br>`HardwareWithBenchmarks()`, `SetHardwareWithBenchmarks()`. |
| **Generic key/value helpers** | `Save(key string, value interface{})` and `Load(key string) (interface{}, error)` allow arbitrary key/value operations on the BoltDB store.  These are used internally by the typed accessors above. |
| **KeyedStorage wrapper** | A small struct that couples a string key with a `*Storage`.  It forwards calls to the underlying storage so callers can work with a “keyed” view: <br>`NewKeyedStorage(key string, s *Storage)`<br>`Save()` and `Load()`. |

---

## Environment variables / flags / command‑line arguments that can be used for configuration  

| Variable / flag | Default value | Purpose |
|------------------|---------------|---------|
| `STORAGE_ENDPOINT` (or `-endpoint`) | `/var/lib/sonm/worker.boltdb` | Path to the BoltDB file. |
| `STORAGE_BUCKET` (or `-bucket`) | `sonm` | Bucket name used by the store. |
| `STORAGE_LOG_LEVEL` (optional) | `info` | Logging level for Zap logger. |

These values are read from a configuration struct (`StorageConfig`) that is passed to `makeStore()`.

---

## Project package structure  

```
insonmnia/
├── state/
│   └── state.go
└── go.mod
```

The only source file in this package is `state.go`.  All other files (e.g. `go.mod`, any tests, or a main package) are outside the scope of this summary.

---

## How the application can be launched if it is a cmd/cli/main package  

1. **Build**  
   ```bash
   go build -o bin/state ./insonmnia/state
   ```
2. **Run** (environment variables)  
   ```bash
   STORAGE_ENDPOINT=/var/lib/sonm/worker.boltdb \
   STORAGE_BUCKET=sonm \
   ./bin/state
   ```

3. **CLI flags** – if the main package uses `flag` or a CLI framework, it can accept:  
   * `-endpoint string` – overrides `STORAGE_ENDPOINT`.  
   * `-bucket string` – overrides `STORAGE_BUCKET`.  

The main package should call `state.NewState()` to obtain a ready‑to‑use storage instance and then use the typed accessors (`PassedBenchmarks()`, etc.) or the generic helpers (`Save/Load`) as needed.

---

## Relations between code entities  

* `Storage` owns a `store.Store` (from Docker libkv) that is created by `makeStore()` using the config values.  
* The in‑memory representation `stateJSON` lives inside `Storage`.  All typed accessors read/write to this struct and then call `dump()`/`loadInitial()` to persist the whole blob.  
* `KeyedStorage` simply holds a key string and forwards all calls to its underlying `*Storage`; it is useful when callers want to work with a specific key/value pair without repeatedly passing the key.

---

## Edge cases / potential dead code  

The file contains no obvious unused functions; every exported method (`NewState`, `Save`, `Load`, etc.) is referenced by at least one other method.  If any of the typed accessors are not used elsewhere, they could be considered dead code, but that would require inspecting the rest of the repository.

---