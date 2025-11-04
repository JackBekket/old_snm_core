# `state` Package Summary

This package manages persistent application state using a key-value store (libkv/boltdb). It handles loading, saving, and accessing benchmark results, hardware information, and associated hashes. The primary goal is to ensure that valid state exists at all times by creating an empty state if none is found on disk during initialization.

**Project Package Structure:**

```
insonmnia/
└── state/
    └── state.go
```

**Configuration:**

*   **Environment Variables / Flags / Cmdline Arguments:** None explicitly defined in the code, but configuration is driven by `StorageConfig`.
*   **Files & Paths:**
    *   `StorageConfig`: Defines storage endpoint and bucket name (default: `/var/lib/sonm/worker.boltdb`, "sonm").  The endpoint path determines where the boltdb file will be stored.

**Edge Cases / Launch Conditions:**

*   If the configured key-value store is inaccessible or corrupted, the application may fail to load initial state and potentially crash if it relies on this data immediately.
*   The immediate persistence of an empty state upon initialization ensures that even in a clean environment, there will always be valid (though initially empty) state available.

**Relations Between Entities:**

*   `Storage`: The central component managing the key-value store connection and state access.
*   `KeyedStorage`: A scoped interface for accessing specific keys within `Storage`.
*   `stateJSON`: Holds the actual application state (benchmarks, hardware info, hash).  This is serialized to JSON for persistence.

**Unclear Places / Dead Code:** None apparent in this single file summary. The code appears focused and functional.