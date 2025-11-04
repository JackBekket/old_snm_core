```markdown
## Package: `cmd` Summary

This package defines the command-line interface (CLI) for an application, likely part of a larger system like Sonm Core. It uses Cobra to create and manage commands, handling configuration loading, version display, and error reporting. The core functionality revolves around setting up a persistent flag for config file path and running a provided function with that context.

**Imports:**

*   `fmt`: For formatted I/O operations (printing errors, version strings).
*   `os`: For interacting with the operating system (exit codes, stdout).
*   `path`: For manipulating file paths (extracting base name of executable).
*   `strings`: For string manipulation (capitalizing error messages).
*   `github.com/mitchellh/go-homedir`: To expand `~` in config paths to the user's home directory.
*   `github.com/sonm-io/core/insonmnia/version`: Accessing application version information.
*   `github.com/sonm-io/core/util`: For platform name retrieval (used in version string).
*   `github.com/spf13/cobra`: The core library for building CLI applications.
*   `github.com/spf13/pflag`: Used to manage flags within Cobra commands.

**External Data / Input Sources:**

*   Command-line arguments (via `os.Args`).
*   Configuration file path specified via the `--config` flag.
*   Environment variables are not directly used, but the expanded home directory (`~`) relies on environment setup.

**TODOs:**

No explicit TODO comments found in this code snippet.

### Code Sections Summary:

1.  **Global Variables & Structures:**
    *   `app`: An `AppContext` struct holding application metadata (name, version, config path). Initialized with the executable name and version from a separate package (`insonmnia/version`).
    *   `showVersion`: A boolean flag to control whether the version is printed.

2.  **`RunWithHomeExpand` Function:**
    *   Expands the `ConfigPath` using `homedir.Expand`, replacing `~` with the user's home directory if present. Handles potential errors during expansion. This ensures that relative paths in config files work correctly.

3.  **`NewCmd` Function:**
    *   Creates a Cobra command (`cobra.Command`) configured to run a provided `Runner` function within an `AppContext`.
    *   Sets up pre-run hooks: checks for the `--version` flag and exits if present, validates required flags (specifically `--config`).
    *   Handles errors during execution by printing capitalized error messages and exiting with code 1.

4.  **Utility Functions:**
    *   `capitalize`: Capitalizes the first letter of a string. Used to format error messages for better readability.
    *   `configFlagHelp`, `versionFlagHelp`: Provide help text for command-line flags.
    *   `versionString`: Formats and returns the application version string, including platform name from `util.GetPlatformName()`.
    *   `checkRequiredFlags`: Iterates through all persistent flags to check if any required flag is missing.

5.  **Flag Handling:**
    *   The `--config` flag is marked as mandatory using `c.MarkPersistentFlagRequired("config")`, ensuring the application cannot run without a valid configuration file path. The `--version` flag displays version information and exits.

### Project Package Structure:

```
cmd/
├── cobra.go
└── common.go
```

### Edge Cases & Launching:

The main entry point is `main.go`.  The application requires the `--config` flag to be set, otherwise it will exit with an error. The `--version` flag can be used to print version information and exit. No other special launch conditions are apparent from this code snippet.
```