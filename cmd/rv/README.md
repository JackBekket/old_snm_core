```markdown
## Package Summary: `main`

This package implements a Rendezvous server using the `github.com/sonm-io/core/insonmnia/npp/rendezvous` library. It handles configuration loading, logger initialization, TLS certificate rotation, and server startup with graceful shutdown handling. The main function executes the `start` command via `cmd.NewCmd`.

**Imports:**

*   `context`: For managing context lifecycle.
*   `fmt`: For formatted printing.
*   `github.com/noxiouz/zapctx/ctxlog`: Structured logging with context awareness.
*   `github.com/sonm-io/core/cmd`: Command execution framework.
*   `github.com/sonm-io/core/insonmnia/logging`: Logger building utilities.
*   `github.com/sonm-io/core/insonmnia/npp/rendezvous`: Rendezvous server implementation.
*   `github.com/sonm-io/core/util`: Utility functions, including certificate rotation.
*   `golang.org/x/sync/errgroup`: For concurrent error handling.

**External Data / Input Sources:**

*   Configuration file path: Loaded from `app.ConfigPath`. The configuration is expected to define logging settings and TLS private key location.
*   TLS Private Key: Used for certificate rotation, loaded from the config.

**Major Code Parts Summary:**

### Configuration & Logger Initialization

The `start` function first loads a server configuration using `rendezvous.NewServerConfig`. It then builds a logger instance based on the logging settings in the configuration file via `logging.BuildLogger`. The context is enriched with the created logger using `log.WithLogger`.

### TLS Certificate Rotation

A hitless certificate rotator (`util.NewHitlessCertRotator`) is initialized to manage TLS certificates, ensuring continuous availability during rotation. This component requires a private key from the configuration and handles automatic renewal of certificates. The rotator is deferred closed for cleanup.

### Rendezvous Server Creation & Startup

The `rendezvous.NewServer` function creates the server instance with options including credentials (TLS config), QUIC support, and logger integration.  An error group (`errgroup.WithContext`) manages concurrent execution of the server's run loop and interruption handling using `cmd.WaitInterrupted`. The server is started within a goroutine managed by the error group.

### Graceful Shutdown & Error Handling

The `wg.Wait()` call blocks until either the server stops or an interrupt signal is received. If an error occurs during shutdown, it's logged before exiting.  The function returns nil on successful completion. The main function simply executes this start command using the cmd framework.

**TODO Comments:**

There are no TODO comments in the provided code snippet.
```