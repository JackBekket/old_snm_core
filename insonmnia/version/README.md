# Package `version`

## Short summary  
The **`version`** package implements a lightweight configuration manager that stores the current platform version in a YAML file under the user’s home directory (`~/.sonm/version.yaml`).  
It fetches the latest GitHub tag via the GitHub API, compares it with the locally stored value, and updates the file if the local copy is stale.  The package exposes an `Observer` interface for reporting validation results and a `ValidateVersion` helper that can be called from a CLI or other packages.

---

## Project structure  

```
insonmnia/
└─ version/
   ├─ version.go
   └─ version_test.go
```

* **`version.go`** – core implementation (types, constants, I/O logic).  
* **`version_test.go`** – unit‑test suite exercising all public methods.

---

## Environment variables / configuration values  

| Variable | Description |
|----------|-------------|
| `versionFilePath` | Path to the YAML file that stores the current version (`~/.sonm/version.yaml`). |
| `versionURI` | GitHub API endpoint used by the default `VersionFetcher`. |

These constants are exported implicitly via the package; they can be overridden by passing options when constructing a `VersionManager`.

---

## Flags / command‑line arguments  

The package itself does not expose any CLI flags, but it provides a public function:

```go
func ValidateVersion(ctx context.Context, observer Observer)
```

which can be invoked from a main program.  The only runtime configuration is the `Observer` implementation (e.g., `LogObserver`) that receives validation events.

---

## Files and their paths  

| File | Path |
|------|------|
| `version.go` | `insonmnia/version/version.go` |
| `version_test.go` | `insonmnia/version/version_test.go` |

Both files belong to the same Go package (`package version`) and are compiled together.

---

## Core entities & their relations  

| Entity | Purpose | Notes |
|--------|---------|-------|
| **`VersionFetcher`** (interface) – strategy for obtaining a semantic version from an external source.  Implemented by `versionFetcher`. |
| **`Observer`** – callback interface used by the manager to report errors and comparison results.  Concrete implementation: `LogObserver`. |
| **`File` / `FS`** – abstractions over file handles; allow swapping out a real or mock filesystem in tests. |
| **`fs`** – concrete `FS` that opens/creates the YAML file at `versionFilePath`. |
| **`options`** – configuration struct passed to `NewVersionManager`; holds default clock, fetcher, and initial version. |
| **`VersionManager`** – orchestrates reading the local file, fetching the latest tag, comparing versions, updating if needed, and persisting back to disk.  It exposes: `ValidateCurrentVersion`, `latestVersion`, `versionData`, `isExpired`. |
| **`versionData` / `versionRawData`** – internal representation of the YAML content; marshaled/unmarshaled by `yaml.v2`. |

The flow is:

1. `ValidateVersion` expands the home directory, creates a `fs{path}` instance and passes it to `NewVersionManager`.
2. The manager reads the file via `versionData()`, fetches the latest tag with its `VersionFetcher`, compares timestamps (`isExpired`) and writes back if stale.
3. Errors are reported through the supplied `Observer`.

---

## Edge cases & launch scenarios  

| Scenario | How to trigger | Expected outcome |
|----------|-----------------|-------------------|
| **File missing** – `ValidateCurrentVersion` creates a new file, writes the current version, and reports via `OnError`.  Tested by `TestVersionManager`. |
| **File open fails** – error is wrapped as an `OSError`; tested by `TestVersionManagerErrFileFailedToOpen`. |
| **Read/parse errors** – handled in `versionData()`; covered by `TestVersionManagerErrFileRead` and `TestVersionManagerErrFileInvalidFormat`. |
| **Missing fields** – missing `"version"` or `"updated"` keys are reported as a `CodecError`; tested by the corresponding unit tests. |
| **Stale version** – if the stored timestamp is older than the configured expiration duration, the manager updates the file and reports via `OnDeprecatedVersion`/`OnBleedingEdgeVersion`.  Covered by `TestVersionManagerErrUpdateWhenExpired*`. |

The package can be launched from a CLI or other code simply by calling:

```go
import "insonmnia/version"

func main() {
    ctx := context.Background()
    observer := version.NewLogObserver(zap.SugaredLogger)
    version.ValidateVersion(ctx, observer)
}
```

---

## Unclear places / potential dead code  

* The `newOptions()` helper is defined but never used in the tests; it could be removed or integrated into `NewVersionManager`.  
* No explicit error handling for HTTP failures beyond a simple `err` return; adding retry logic would improve robustness.  

No obvious dead code was found.

---