```markdown
## Package/Component Summary: `main` (Oracle Service Entrypoint)

**Package Name:** `main`

**Imports:**
*   `context`: For managing goroutine lifecycles and cancellation.
*   `fmt`: For formatted I/O, primarily error reporting.
*   `golang.org/x/sync/errgroup`: For concurrent execution with context-aware error handling.
*   `github.com/sonm-io/core/cmd`: Custom command framework for application setup and execution.
*   `github.com/sonm-io/core/insonmnia/logging`: Logging utilities.
*   `github.com/sonm-io/core/insonmnia/oracle`: Oracle configuration and service logic.

**External Data / Input Sources:**
*   Configuration file path: Provided via `app.ConfigPath` (presumably from the command framework). This config is parsed by `oracle.NewConfig`.
*   Command line arguments: Handled through the `cmd` package, though not explicitly visible in this snippet.

**TODOs:** None found in provided code.

---

### Code Summary Sections:

**1. Application Entrypoint (`main`)**: The `main` function initializes and executes a command using the `cmd` framework. This is standard boilerplate for Sonm Core applications, providing a structured way to handle application lifecycle (setup, execution, teardown).

**2. Configuration & Logging Initialization (`run`)**:  The `run` function first loads configuration from a file path provided by the application context (`app.ConfigPath`). It then builds a logger instance based on logging settings within the config. Error handling is present for both operations; failures result in immediate termination with formatted error messages.

**3. Oracle Instance Creation & Service Execution**: An `oracle.NewOracle` function creates an oracle service instance, passing context and configuration. The core logic involves launching two concurrent goroutines using `errgroup`. One waits for interruption signals (likely SIGINT/SIGTERM), while the other runs the Oracle's main serving loop (`o.Serve(ctx)`).

**4. Concurrent Execution & Termination Handling**:  The `errgroup` ensures that both goroutines run concurrently, and any error from either will cause the entire process to terminate gracefully after waiting for all running routines. The final return statement indicates successful completion if no errors occurred during execution or shutdown.
```