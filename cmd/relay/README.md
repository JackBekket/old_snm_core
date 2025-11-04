## Package: `relay`

This package implements a relay server application. It loads configuration from a file, initializes logging, and starts the relay server. The server runs until an interrupt signal is received.

**Project Package Structure:**

```
cmd/relay/
├── main.go
```

**Configuration:**

*   **Configuration File:** The server configuration is loaded from a file specified by `app.ConfigPath` within the `cmd.AppContext`.
*   **Logging Configuration:** Logging is configured using the `cfg.Logging` structure.

**Environment Variables/Flags/Cmdline Arguments:**

*   The application uses `cmd.NewCmd` which suggests it accepts standard command-line flags defined within the `cmd` package. Specific flags are not visible in the provided snippet.

**Edge Cases (Launch):**

*   The application can be launched directly via `go run cmd/relay/main.go`.
*   The `cmd.NewCmd` function suggests the application can be launched with command-line arguments, but the exact arguments are not specified in the provided code.

**Code Relations:**

*   `main.go` serves as the entry point, initializing the application using `cmd.NewCmd`.
*   The `start` function handles the core logic of loading configuration, initializing logging, creating the relay server, and running it.
*   The `relay.NewServerConfig` function is used to create the server configuration from the loaded file.
*   The `logging.BuildLogger` function is used to create the logger instance.
*   The `errgroup.Group` is used to manage concurrent execution of the server and signal handling.