# cmd/connor/main.go  
**Package Name:** `main`  
  
**Imports:**  
- `fmt` (standard library for formatted I/O)  
- `github.com/noxiouz/zapctx/ctxlog` (context logging utility)  
- `github.com/sonm-io/core/cmd` (command execution framework)  
- `github.com/sonm-io/core/connor` (configuration and server initialization)  
- `github.com/sonm-io/core/insonmnia/logging` (custom logging implementation)  
- `github.com/sonm-io/core/util/metrics` (Prometheus metrics exporter)  
- `golang.org/x/net/context` (context management)  
- `golang.org/x/sync/errgroup` (error handling with goroutines)  
  
**External Data / Input Sources:**  
- **Configuration File Path:** The application expects a configuration file path via the command line or environment variables, used by `connor.NewConfig`.  The config is loaded from this path and drives most of the application's behavior.  
- **Environment Variables:** Configuration values can also be sourced from environment variables (implicitly through `connor`).  
  
**TODO Comments:** None found in provided code snippet.  
  
---  
  
### Main Execution Flow:  
  
The `main` function initializes a command using `cmd.NewCmd(run)` and executes it. The `run` function is the core logic of this file. It loads configuration, sets up logging, and then launches three concurrent goroutines managed by an `errgroup`.  These routines handle signal interruption waiting, server initialization/serving (using `connor`), and Prometheus metrics serving.  
  
### Configuration Loading:  
  
The code uses `connor.NewConfig` to load a configuration from the specified path (`app.ConfigPath`). Errors during config loading are fatal. The loaded configuration is then used throughout the application for various settings, including logging and server parameters.  
  
### Logging Setup:  
  
A logger instance is created using `logging.BuildLogger`, configured based on the log settings in the loaded configuration file (`cfg.Log`).  The context is enriched with this logger via `ctxlog.WithLogger`.  
  
### Concurrent Execution & Error Handling:  
  
An `errgroup` ensures that all three goroutines (signal handling, server serving, metrics exporting) either complete successfully or terminate if any one fails. The `wg.Wait()` call blocks until all routines finish or an error occurs.  The application exits with a formatted error message if the errgroup encounters issues during termination.  
  
### Server & Metrics Serving:  
  
- A `connor` instance is created and served using `server.Serve(ctx)`.  
- Prometheus metrics are exported via `metrics.NewPrometheusExporter`, configured from `cfg.Metrics` and with logging enabled.  The exporter also serves on a specified port (defined in the config).  
  
