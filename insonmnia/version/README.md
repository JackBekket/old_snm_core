# `version` Package Summary

**Package Name:** `version`

This package manages application version validation against a remote source (GitHub API). It reads the current version from `~/.sonm/version.yaml`, fetches latest tags from GitHub, and compares them to determine if an update is needed.  Observers can be registered for notifications about outdated versions. The core logic resides in `VersionManager` which handles fetching, storing, and comparing versions with configurable options (clock, fetcher, version override).

**Configuration:**

*   **Environment Variables:** None explicitly used in the code.
*   **Flags/Cmdline Arguments:** No direct command-line arguments are present; configuration is file-based or via `Option` function parameters.
*   **Files:**
    *   `~/.sonm/version.yaml`: Stores current version (YAML format). Created if missing.
    *   GitHub API endpoint: `https://api.github.com/repos/sonm-io/core/git/refs/tags`. Used for fetching latest tags.

**Project Package Structure:**

```
insonmnia/version/
├── version.go  (Core logic, VersionManager, Fetcher)
└── version_test.go (Unit tests with mocking)
```

**Relations between Entities:**

*   `VersionManager`: Orchestrates the entire process: fetching, storing, comparing versions.
*   `VersionFetcher`: Fetches latest tags from GitHub API.  Can be mocked for testing.
*   `FS`: Abstraction over filesystem operations (reading/writing `version.yaml`). Mockable for tests.
*   `Clock`: Provides time-dependent behavior (expiration checks). Also mockable.

**Edge Cases:**

The application can be launched without a version file, in which case it will create one with the default hardcoded version ("v0.4.0-f6461c2a").  If GitHub API is unreachable or returns invalid data, errors are handled gracefully (though tests suggest reliance on correct API responses). The package does not explicitly handle concurrent access to `version.yaml`, which could lead to race conditions if multiple instances try to update it simultaneously.

<br>