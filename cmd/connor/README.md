# connor

This package implements a command-line application that loads configuration, sets up logging, starts a server using `connor`, and exports Prometheus metrics. The main execution flow involves concurrent goroutines for signal handling, server serving, and metric exporting managed by an error group to ensure proper termination. Configuration is loaded from a file path or environment variables via the `connor` package.

**Configuration:**
- **Config Path:** Specified through command line arguments (not explicitly shown in snippet) or environment variable (`app.ConfigPath`). The application expects a configuration file at this location.
- **Environment Variables:** Configuration values can be overridden by setting corresponding environment variables.
- **Metrics Config:** Prometheus metrics are configured via the `cfg.Metrics` section of the loaded config, including port and other settings.

**Files & Structure:**

```
cmd/connor/
├── main.go
```

**Relationships:**

The `main.go` file orchestrates all components: configuration loading (`connor.NewConfig`), logging setup (`logging.BuildLogger`, `ctxlog.WithLogger`), server initialization and serving (`server.Serve`), and metrics exporting (`metrics.NewPrometheusExporter`). The `errgroup` ensures that these operations either complete successfully or terminate cleanly if any one fails.

**Edge Cases:**
- If the configuration file cannot be loaded, the application exits with a fatal error.
- Errors during server serving or metric export will cause the entire application to exit due to the use of an `errgroup`.