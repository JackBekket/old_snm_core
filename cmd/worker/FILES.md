# cmd/worker/main.go  
## Package: main  
  
**Imports:**  
  
*   `context`: For managing context and cancellation.  
*   `fmt`: For formatted I/O.  
*   `os`: For operating system functionalities.  
*   `os/signal`: For handling OS signals.  
*   `syscall`: For system calls.  
*   `github.com/noxiouz/zapctx/ctxlog`: For structured logging with context.  
*   `github.com/sonm-io/core/cmd`: For command-line application structure.  
*   `github.com/sonm-io/core/insonmnia/logging`: For custom logging setup.  
*   `github.com/sonm-io/core/insonmnia/state`: For state management.  
*   `github.com/sonm-io/core/insonmnia/version`: For version validation.  
*   `github.com/sonm-io/core/insonmnia/worker`: For the core worker functionality.  
*   `github.com/sonm-io/core/util/metrics`: For exporting metrics.  
*   `go.uber.org/zap`: For structured logging.  
*   `go.uber.org/zap/zapcore`: For customizing Zap logger cores.  
*   `golang.org/x/sync/errgroup`: For managing concurrent goroutines with error handling.  
  
**External Data/Input Sources:**  
  
*   **Configuration File:** Loaded via `worker.NewConfig(app.ConfigPath)`. The path is provided through the `app.ConfigPath` which is likely passed from the command-line application context.  
*   **Command-Line Arguments:** Handled by `cmd.NewCmd(run).Execute()`, providing the application context (`app`).  
*   **Environment Variables:** Potentially used within the configuration file or by the worker itself.  
*   **OS Signals:** Listens for `SIGINT` and `SIGTERM` to gracefully shut down.  
  
**TODOs:**  
  
*   No explicit `TODO` comments found in the provided code.  
  
### Code Summary  
  
**1. Application Entry Point:**  
  
The `main` function initializes the command-line application using `cmd.NewCmd(run).Execute()`. This sets up the application context and executes the `run` function.  
  
**2. Configuration Loading & Logging:**  
  
The `run` function loads the configuration from a file specified by `app.ConfigPath` using `worker.NewConfig`. It then sets up a custom Zap logger with a watcher core for logging events. The logger is attached to the context using `log.WithLogger`. Version validation is performed using `version.ValidateVersion`.  
  
**3. State Management:**  
  
A state storage is created using `state.NewState`, initialized with the configuration's storage settings.  
  
**4. Signal Handling:**  
  
A goroutine is launched to listen for `SIGINT` and `SIGTERM` signals. When received, it logs a message and cancels the context, triggering a graceful shutdown.  
  
**5. Worker Initialization & Serving:**  
  
The core worker is initialized using `worker.NewWorker`, passing the configuration, state storage, context, version, and log watcher. The worker's `Serve` method is called to start the worker's main loop.  
  
**6. Metrics Exporting:**  
  
A Prometheus exporter is started in a separate goroutine using `metrics.NewPrometheusExporter`. It listens on the address specified in the configuration and exports metrics.  
  
**7. Error Handling & Shutdown:**  
  
If the worker encounters an error during serving, the context is canceled, an error is logged, and the program waits for all goroutines to finish using `waiter.Wait()`.  
  
