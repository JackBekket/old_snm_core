```markdown
## Package: sonmmon

**Summary:**

The `sonmmon` package is a command-line utility designed to monitor the status of a SONM worker node. It connects to a gRPC server to retrieve worker information, displays it in a graphical window using SDL2, and allows the user to switch between the graphical display and the terminal using Alt+F1. The application relies heavily on external configuration files, environment variables, and system commands to determine its behavior.

**Project Package Structure:**

```
cmd/sonmmon/
├── TerminusTTFWindows-4.46.0.ttf
├── image.png
└── main.go
```

**Configuration:**

*   **Environment Variables:**
    *   `SONM_USER`: Used to determine the user's home directory if not explicitly set.
*   **Configuration File:**
    *   `cli.yaml`: Located in the user's home directory (determined by `guessHomeDir`) or `/home/sonm/.sonm/cli.yaml` as a fallback. Contains configuration settings for the gRPC server address and other parameters.
*   **Keystore:**
    *   Used for authentication with the gRPC server. The exact location is not explicitly defined in the code but is likely specified in the `cli.yaml` configuration.
*   **gRPC Server:**
    *   Connects to `127.0.0.1:15030` by default. This can be overridden via the `cli.yaml` configuration.

**Command-Line Arguments/Flags:**

The application does not appear to take any explicit command-line arguments or flags. All configuration is loaded from the `cli.yaml` file and environment variables.

**Edge Cases/Launch Conditions:**

*   **X Server Requirement:** The application attempts to start an X server if the `DISPLAY` environment variable is not set. This may cause issues on headless systems or if an X server is not available.
*   **Dependency on External Commands:** The application relies on the `pgrep` command to find the SONM node process. If `pgrep` is not installed or not in the system's PATH, the application may fail.
*   **Configuration File Errors:** If the `cli.yaml` file is missing or contains invalid configuration, the application may crash or behave unpredictably.
*   **gRPC Connection Failures:** If the gRPC server is unreachable or authentication fails, the application will not be able to retrieve worker status and will likely exit.

**Code Relations & Unclear Places:**

*   The `guessHomeDir` function attempts to determine the user's home directory using multiple fallback mechanisms. This logic could be simplified or made more robust.
*   The `initGraphics` function initializes SDL2 and loads the background image. The image path is hardcoded, which may limit flexibility.
*   The `main` function's loop includes a `TODO` comment suggesting asynchronous signal handling. This indicates potential future improvements to the application's responsiveness.
*   The `displayCtl` struct manages the SDL2 window and rendering. The code could be refactored to improve readability and maintainability.

<end_of_output>
```