## Package: `connor`

**Project Package Structure:**

```
cmd/connor/
├── main.go
```

**Summary:**

The `connor` package represents a command-line application that launches a server, exposes Prometheus metrics, and handles graceful shutdown. It loads configuration from a file, initializes logging, and manages concurrent goroutines using an `errgroup`. The application waits for an interruption signal to terminate.

**Configuration:**

*   **Configuration File Path:** Determined by `app.ConfigPath` from `cmd.AppContext`.
*   **Configuration Structure:** Loaded using `connor.NewConfig`. Contains settings for logging and metrics.

**Environment Variables/Flags/Cmdline Arguments:**

*   None explicitly defined in the provided code snippet. Configuration is loaded from a file.

**Edge Cases (Launch):**

*   The application exits if the configuration file cannot be loaded.
*   The application exits if the logger cannot be initialized.
*   The application exits if the Connor server fails to start.
*   The application exits if the Prometheus metrics exporter fails to start.

**Code Relations:**

*   `main.go` orchestrates the entire application lifecycle.
*   `connor.NewConfig` loads configuration.
*   `logging.BuildLogger` initializes logging.
*   `connor.New` creates the Connor server instance.
*   `metrics.NewPrometheusExporter` sets up Prometheus metrics.
*   `cmd.WaitInterrupted` handles graceful shutdown.
*   `errgroup` manages concurrent goroutines and error handling.