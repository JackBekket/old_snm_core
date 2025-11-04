```markdown
# cmd/oracle:main

## Summary

This package serves as the entry point for the Oracle service within the Sonm Core ecosystem. It initializes configuration, logging, and an oracle instance before launching a concurrent serving loop that runs until interrupted. The application uses `errgroup` to manage goroutines and ensure graceful shutdown in case of errors. Configuration is loaded from a file path specified via command-line arguments or environment variables (see below).

## Environment Variables / Flags

*   **CONFIG_PATH**: Path to the Oracle configuration file. If not provided, defaults to `/etc/sonm/oracle.yml`.
*   **LOG_LEVEL**: Logging level (e.g., `debug`, `info`, `warn`, `error`). Defaults to `info`.
*   **LOG_FORMAT**: Log output format (`text` or `json`). Defaults to `text`.

## Files and Paths

*   `main.go`: The main entry point for the Oracle service.
*   Configuration file: Loaded from the path specified by the `CONFIG_PATH` environment variable or `/etc/sonm/oracle.yml` if not set.

## Launch Edge Cases

The application can be launched directly using `go run main.go`. Configuration is loaded from the default location unless overridden via command-line arguments or environment variables. The service will exit immediately if configuration loading fails.  If no errors occur, it runs indefinitely until interrupted by a signal (e.g., SIGINT/SIGTERM).

## Project Package Structure

```
cmd/oracle/
├── main.go
```

## Code Relations and Unclear Places

The code relies heavily on the `github.com/sonm-io/core` internal packages, particularly `insonmnia/logging` and `insonmnia/oracle`. The exact implementation details of these dependencies are not visible in this snippet but are crucial for understanding how configuration is parsed, logging is handled, and the Oracle service operates internally.  The `o.Serve(ctx)` function within the serving loop likely handles incoming requests or performs background tasks related to oracle functionality (e.g., data retrieval, price updates).