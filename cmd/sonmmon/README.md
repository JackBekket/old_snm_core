## Package Summary: `main`

This package implements a command-line tool for monitoring and displaying SONM worker status, primarily targeting Linux environments. It leverages gRPC communication with a SONM node to retrieve real-time metrics such as resource usage (RAM, storage, network), income, uptime, and overall worker health. The application also includes basic graphics rendering using SDL2 to visualize this data on the screen.

**Imports:**

*   `crypto/ecdsa`: For key management related to SONM authentication.
*   `github.com/ethereum/go-ethereum/crypto`: Ethereum crypto utilities for address generation.
*   `github.com/sonm-io/core/...`: Core SONM libraries for gRPC communication, configuration handling, and data structures.
*   `github.com/veandco/go-sdl2/...`: SDL2 bindings for graphics rendering.
*   `golang.org/x/sync/errgroup`: For managing concurrent operations with error handling.
*   `google.golang.org/grpc`: gRPC library for communication with the SONM node.

**External Data Sources:**

*   Configuration file (`cli.yaml`) in `$HOME/.sonm`.  Falls back to `/home/sonm/.sonm/cli.yaml` if home directory cannot be determined.
*   SONM worker node via gRPC connection (default address: `127.0.0.1:15030`).

**TODOs:**

*   The comment "// todo: maybe move to goroutine and wait for signal asynchronously?" suggests a potential refactoring of the event handling loop to improve responsiveness or efficiency.
*   The note "note: not sure with this solution, maybe we should instantly terminate" indicates uncertainty about fallback behavior when config path cannot be determined.

### Code Sections Summary:

**1. Home Directory Detection (`guessHomeDir`, `guessHomeViaEnv`, `guessHomeViaProc`):**

These functions attempt to determine the user's home directory using environment variables, process information (specifically looking for a running `sonmnode` process), or fallback mechanisms if neither succeeds.  The logic prioritizes reliability by checking multiple sources before giving up.

**2. Configuration Path Resolution (`guessConfigPath`):**

This function constructs the path to the configuration file based on the detected home directory and default SONM config location. It handles potential errors in determining the home directory gracefully, falling back to a hardcoded path if necessary.

**3. Worker Status Management (`WorkerStatus`, `NewWorkerStatus`, `update`):**

The `WorkerStatus` struct encapsulates all relevant metrics about the worker node. The `update` method retrieves this data via gRPC calls and populates the structure accordingly, handling potential communication errors.  It also calculates derived values like resource utilization percentages.

**4. gRPC Client Creation (`newClient`):**

This function establishes a secure gRPC connection to the SONM worker node using TLS authentication based on an ECDSA key loaded from the configuration file.

**5. Graphics Initialization and Rendering (`initGraphics`, `displayCtl`, `drawText`):**

The code initializes SDL2, loads a background image, sets up a window, and defines functions for drawing text onto the screen.  It handles potential errors during initialization and provides basic rendering capabilities. The graphics are designed to display worker status in real-time.

**6. Main Loop (`main`):**

This function orchestrates the entire process: loading configuration, establishing gRPC connection, initializing graphics, fetching worker status periodically, updating the screen with live data, and handling user input (specifically Alt+F1 for switching TTY). The loop continues until an error occurs or a quit event is received.  The main loop also includes logic to switch to tty7 on alt-f1 keypress.

**Build Tags:**

*   `// +build linux`: This indicates that the package is specifically compiled only when building for Linux systems.

### Project Package Structure:

```
cmd/sonmmon/
├── main.go
├── TerminusTTFWindows-4.46.0.ttf
└── image.png
```