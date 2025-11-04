# `sonmmon` Package Summary

This package implements a command-line tool for monitoring and displaying SONM worker status, primarily targeting Linux environments. It leverages gRPC communication with a SONM node to retrieve real-time metrics such as resource usage (RAM, storage, network), income, uptime, and overall worker health. The application also includes basic graphics rendering using SDL2 to visualize this data on the screen.

**Environment Variables:**

*   `HOME`: Used for determining the user's home directory if not found via other means.
*   `SONM_CLI_CONFIG`: Overrides default config path, allowing custom configuration location.

**Cmdline Arguments/Flags:**

None explicitly defined in provided code summary; relies on hardcoded defaults and environment variables.

**Files & Paths (Configuration):**

*   `$HOME/.sonm/cli.yaml` (primary)
*   `/home/sonm/.sonm/cli.yaml` (fallback if `$HOME` is unavailable or invalid).

**Edge Cases:**

1.  If the home directory cannot be determined, it falls back to `/home/sonm/.sonm/cli.yaml`. The comment "note: not sure with this solution, maybe we should instantly terminate" suggests uncertainty about this fallback behavior.
2.  The application is built only for Linux (`// +build linux`). Attempting to build on other platforms will result in compilation errors.

**Project Package Structure:**

```
cmd/sonmmon/
├── TerminusTTFWindows-4.46.0.ttf (Font file, likely used for rendering text)
├── image.png (Background image for the SDL2 window)
└── main.go (Main application logic)
```

**Code Entity Relations:**

*   `main.go` orchestrates all operations: config loading, gRPC client creation, graphics initialization, and real-time status updates.
*   The `WorkerStatus` struct holds worker metrics retrieved via gRPC calls. The `update` method populates this structure.
*   SDL2 functions (`initGraphics`, `displayCtl`, `drawText`) handle the graphical rendering of worker data.

**Unclear Places/Dead Code:**

*   The comment "// todo: maybe move to goroutine and wait for signal asynchronously?" suggests a potential refactoring point in event handling, but it's unclear if this is actively planned or abandoned.
*   The note "note: not sure with this solution, maybe we should instantly terminate" indicates uncertainty about fallback behavior when config path cannot be determined.