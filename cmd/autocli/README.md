# Package: `autocli`

This package serves as the command-line interface entry point for Sonm Core, utilizing the `xcode` utility to execute core operations. It handles errors by printing them to standard output before exiting with a non-zero status code (-1). The `proto/mod.go` file likely contains initialization logic or flag definitions used during execution.

**File Structure:**

*   `main.go`: Entry point for the command-line application.
*   `proto/mod.go`: Contains supporting functions, potentially including flag registration or configuration loading.

**Environment Variables / Flags:**

The code does not explicitly define environment variables or flags within this snippet. However, `xcode.Execute()` likely handles these through its own mechanisms (command-line arguments, config files). The blank import of the `proto` package suggests that it might register command-line flags during initialization.

**Edge Cases:**

The application exits with a non-zero status code (-1) if `xcode.Execute()` returns an error. This indicates potential issues with external configuration or input data, but no specific edge cases are handled within this file itself. The behavior of the program depends entirely on how `xcode.Execute()` handles errors and invalid inputs.

**Code Summary:**

The `main` function calls `xcode.Execute()`, which performs the core application logic. Any errors during execution are printed to standard output, and the program exits with a non-zero status code (-1). The blank import of `github.com/sonm-io/core/cmd/autocli/proto` suggests that it might register command-line flags or perform other initialization tasks before execution.