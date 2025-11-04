# `version` Package Summary

The `version` package manages application version validation and updates. It fetches the latest version from a GitHub API, stores it in a YAML file (`~/.sonm/version.yaml`), and compares it against the current version. The package uses an observer pattern to notify external components about version status (outdated, unstable).

**Configuration:**

*   **Environment Variable:** `Version` (initial application version).
*   **YAML File:** `~/.sonm/version.yaml` (stores latest version and update timestamp).
*   **GitHub API:** `https://api.github.com/repos/sonm-io/core/git/refs/tags` (source for latest version).
*   **Options:** `VersionManager` can be configured with custom clocks and version fetchers via `Option` type.

**Edge Cases:**

The package handles file system errors, invalid YAML format, missing version data, and version mismatches. It uses custom error types for specific failure scenarios.

**Project Structure:**

```
insonmnia/
├── version/
│   ├── version.go
│   └── version_test.go
```

**Relations:**

*   `VersionManager` orchestrates version fetching, validation, and updates.
*   `VersionFetcher` retrieves the latest version from GitHub.
*   `FS` abstracts file system operations for testability.
*   `Observer` interface allows external components to react to version changes.