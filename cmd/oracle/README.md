# Package: `oracle`

**Summary:**

The `oracle` package implements a service that loads configuration, initializes a logger, and serves an oracle instance. It uses concurrent goroutines to handle interruption signals and the oracle service itself. The package relies on external configuration files and command-line arguments for setup.

**Project Package Structure:**

```
cmd/oracle/main.go
```

**Configuration:**

*   **Environment Variables:** None explicitly used in the provided code.
*   **Flags/Cmdline Arguments:** Handled by the `cmd` package, but specific arguments are not defined in this snippet.
*   **Files:**
    *   `app.ConfigPath`: Path to the configuration file loaded by `oracle.NewConfig`.

**Edge Cases:**

*   The application can be launched via the `cmd` package's command-line interface. The exact command-line arguments are not specified in this snippet.
*   The application terminates when an interruption signal is received or when the oracle service encounters an unrecoverable error.

**Relations Between Code Entities:**

*   `main.go` serves as the entry point, initializing the `cmd` application context, loading configuration, creating a logger, and starting the oracle service.
*   The `oracle` package provides the core oracle functionality, including configuration loading and service management.
*   The `logging` package provides logging utilities used by the oracle service.
*   The `errgroup` package manages concurrent goroutines, ensuring that the oracle service runs until interrupted or encounters an error.