## Package: `logging`

This package provides a flexible logging solution built on top of Uber's Zap library, with features like configurable output (stdout/stderr), custom log levels, terminal colorization, OpenTracing integration, and a subscription-based message broadcasting mechanism for external sinks. The core functionality revolves around constructing a `zap.Logger` instance via the `BuildLogger` function, which takes configuration from a `Config` struct (likely loaded from YAML). Log levels are handled using a custom `Level` type with parsing and serialization methods. Trace IDs can be injected into logs if OpenTracing spans exist in the context. The `watcher` component allows external components to subscribe to log messages via channels for real-time processing or forwarding.

**Configuration:**

*   **Environment Variables/Flags:** None explicitly defined, but configuration is expected from a YAML file loaded into the `Config` struct.
*   **Cmdline Arguments:** Not applicable (library package).
*   **Files/Paths:** Configuration files are assumed to be loaded via external mechanisms (e.g., YAML parsing) and not hardcoded in this package.

**Edge Cases:**

The application is a library, so there's no direct launch edge case. However, misconfigured `Config` structs could lead to unexpected behavior:
*   Invalid log levels will result in errors during level parsing.
*   Incorrect output settings (e.g., invalid file paths) may cause logging failures.

**Project Package Structure:**

```
insonmnia/logging/
├── config.go       # Defines the Config struct and related methods for loading configuration.
├── logging.go      # Core logger construction, level handling, terminal detection.
├── logging_test.go # Unit tests for log level parsing.
├── trace.go        # OpenTracing integration: adds trace IDs to logs if available in context.
└── watcher.go      # Subscription-based message broadcasting mechanism for external sinks.
```

**Relations between Code Entities:**

*   `config.go` defines the configuration structure used by `logging.go`.
*   `logging.go` uses custom `Level` type and parsing functions to set log levels in Zap logger.
*   `trace.go` enhances logs with trace IDs if OpenTracing is active, leveraging context information.
*   `watcher.go` provides a mechanism for external components to subscribe to log messages via channels, integrating with the core logging process through `WatcherCore`.