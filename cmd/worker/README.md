# Package: cmd/worker

This package implements the main entry point for a worker application. It handles configuration loading, logging setup, signal handling, worker initialization, and metric exporting. The application gracefully shuts down upon receiving `SIGINT` or `SIGTERM` signals.

**Configuration:**

*   **Config Path:** `app.ConfigPath` (command-line argument) specifies the path to the worker configuration file.
*   **Environment Variables:** The configuration file may use environment variables for dynamic settings.
*   **Metrics Address:** Configured via the configuration file, used by the Prometheus exporter.

**Command-Line Arguments:**

*   The application is launched via `cmd.NewCmd(run).Execute()`, which handles command-line arguments and passes them to the `run` function through the `app` context.

**Files:**

*   `cmd/worker/main.go`: Contains the main application logic, including configuration loading, logging, signal handling, worker initialization, and metric exporting.

**Dependencies:**

*   `github.com/noxiouz/zapctx/ctxlog`: Structured logging with context.
*   `github.com/sonm-io/core/cmd`: Command-line application structure.
*   `github.com/sonm-io/core/insonmnia/logging`: Custom logging setup.
*   `github.com/sonm-io/core/insonmnia/state`: State management.
*   `github.com/sonm-io/core/insonmnia/version`: Version validation.
*   `github.com/sonm-io/core/insonmnia/worker`: Core worker functionality.
*   `github.com/sonm-io/core/util/metrics`: Metric exporting.
*   `go.uber.org/zap`: Structured logging.
*   `golang.org/x/sync/errgroup`: Concurrent goroutine management.

**Edge Cases:**

*   The application exits if the configuration file is invalid or if the worker encounters an error during serving.
*   Graceful shutdown is triggered by `SIGINT` or `SIGTERM` signals.