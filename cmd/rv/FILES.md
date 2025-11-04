# cmd/rv/main.go  
## Package: `main`  
  
**Imports:**  
  
*   `context`: For managing context and cancellation.  
*   `fmt`: For formatted I/O.  
*   `github.com/noxiouz/zapctx/ctxlog`: For structured logging with context.  
*   `github.com/sonm-io/core/cmd`: For command-line application structure.  
*   `github.com/sonm-io/core/insonmnia/logging`: For building logger instances.  
*   `github.com/sonm-io/core/insonmnia/npp/rendezvous`: For Rendezvous server functionality.  
*   `github.com/sonm-io/core/util`: For utility functions, including certificate rotation.  
*   `golang.org/x/sync/errgroup`: For managing concurrent goroutines with error handling.  
  
**External Data/Input Sources:**  
  
*   **Configuration File:** The code loads a configuration file using `rendezvous.NewServerConfig(app.ConfigPath)`. The path to this file is provided via the `app.ConfigPath` variable, which is likely passed from the `cmd` package.  
*   **Private Key:** The code uses a private key (`cfg.PrivateKey`) for TLS certificate generation. The source of this key is not explicitly defined in the snippet but is assumed to be part of the configuration.  
  
**TODOs:**  
  
*   No TODO comments found in the provided code.  
  
---  
  
### Initialization and Configuration  
  
The `start` function initializes the Rendezvous server. It first loads the configuration from a file specified by `app.ConfigPath`. It then builds a logger instance based on the logging configuration within the loaded config. A certificate rotator is created using the private key from the config, ensuring TLS certificate management.  
  
### Server Creation and Execution  
  
The core of the function creates a `rendezvous.Server` instance with TLS credentials, QUIC support, and logging integration. The server is then launched in a goroutine using an `errgroup` to manage concurrent execution. Another goroutine waits for an interruption signal. The `errgroup` ensures that both goroutines complete before the function returns.  
  
### Error Handling and Logging  
  
The code includes robust error handling at each step, wrapping errors with informative messages. Logging is used extensively to track server startup, shutdown, and any errors encountered. The server shutdown is logged with the error that caused it.  
  
### Main Function  
  
The `main` function simply creates a command using `cmd.NewCmd(start)` and executes it. This sets up the command-line interface and runs the `start` function when the application is invoked.  
  
