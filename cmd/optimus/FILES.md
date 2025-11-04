# cmd/optimus/main.go  
## Package: `main`  
  
**Imports:**  
  
*   `context`: For managing context.  
*   `fmt`: For formatted I/O.  
*   `github.com/noxiouz/zapctx/ctxlog`: For structured logging with context.  
*   `github.com/sonm-io/core/cmd`: For command-line application structure.  
*   `github.com/sonm-io/core/insonmnia/version`: For version validation.  
*   `github.com/sonm-io/core/optimus`: Core functionality, config loading, and bot creation.  
*   `go.uber.org/zap`: For structured logging.  
*   `go.uber.org/zap/zapcore`: For zap configuration.  
  
**External Data/Input Sources:**  
  
*   **Configuration File:** Loaded via `optimus.LoadConfig(app.ConfigPath)`. The path to the config file is provided through the `app.ConfigPath` variable, which is part of the `cmd.AppContext`.  
*   **Command-Line Arguments:** Handled by the `cmd` package, which parses arguments and passes them to the `run` function via `cmd.AppContext`.  
*   **Environment Variables:** Potentially used within the configuration file itself.  
*   **App Version:** Passed via `app.Version` to `optimus.NewOptimus`.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Initialization and Configuration  
  
The `main` function initializes the command-line application using the `cmd` package and executes the `run` function. The `run` function loads the configuration from a file specified by `app.ConfigPath` using `optimus.LoadConfig`. It handles potential restrictions defined in the configuration using `optimus.RestrictUsage`.  
  
### Logging Setup  
  
The code configures a `zap` logger based on the logging level specified in the configuration (`cfg.Logging.LogLevel()`). The logger is then attached to the context using `ctxlog.WithLogger`. The logger is configured to output to stdout and stderr with colored level encoding.  
  
### Version Validation  
  
The `version.ValidateVersion` function is called to validate the application version using the configured logger.  
  
### Optimus Bot Creation and Execution  
  
An `Optimus` bot is created using `optimus.NewOptimus`, passing the configuration, application version, and logger. Finally, the bot's `Run` method is called with the context to start the application's core logic.  
  
