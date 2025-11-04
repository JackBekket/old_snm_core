## Package: `rv`

This package implements a Rendezvous server, likely for peer discovery or communication in a distributed system. It handles TLS certificate rotation, QUIC protocol support, and structured logging.

**Configuration:**

*   **`app.ConfigPath` (string):** Path to the server configuration file.  This is likely passed as a command-line argument or environment variable by the `cmd` package.
*   **`cfg.PrivateKey` (string):** Path to the private key used for TLS certificate generation.  Also likely configured via the config file.
*   **Logging Configuration:**  The server's logging behavior is determined by the configuration loaded from `app.ConfigPath`.

**Files:**

```
cmd/rv/
├── main.go
```

**Logic Summary:**

1.  **Initialization:** Loads configuration from `app.ConfigPath`, creates a logger, and sets up a certificate rotator using `cfg.PrivateKey`.
2.  **Server Creation:** Instantiates a `rendezvous.Server` with TLS, QUIC, and logging.
3.  **Execution:** Launches the server in a goroutine managed by an `errgroup`.  Another goroutine waits for an interrupt signal.
4.  **Error Handling:**  Errors are wrapped with context and logged throughout the process.
5.  **Command-Line Interface:** The `main` function uses `cmd.NewCmd(start)` to create a command-line application that executes the `start` function.

**Edge Cases (Launch):**

*   The application can be launched directly via `go run cmd/rv/main.go`.
*   The `app.ConfigPath` must be correctly set, or the server will fail to start.
*   The `cfg.PrivateKey` must be valid, or TLS certificate generation will fail.

**Relations:**

*   `cmd` package provides the command-line interface structure.
*   `insonmnia/logging` provides the logger instance.
*   `insonmnia/npp/rendezvous` implements the core Rendezvous server logic.
*   `util` provides certificate rotation functionality.
*   `context` and `errgroup` manage concurrency and error handling.