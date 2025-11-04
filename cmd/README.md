# Sonm Core CLI Package Summary

This document summarizes the functionality and structure of the `cmd` package within the Sonm Core project. The package is responsible for building a command-line interface (CLI) using Cobra, handling configuration loading, version display, signal interruption, and error reporting. It provides utilities for managing flags, executing commands with context, and gracefully shutting down long-running processes.

## Project Package Structure:

*   `cmd/cobra.go`: Core CLI setup, command registration, flag parsing, and execution logic.
*   `cmd/common.go`: Utility functions for handling signals (SIGINT, SIGTERM) and context cancellation.

## Configuration & Environment Variables:

The application relies on the `--config` flag to specify a configuration file path. The `~` character in this path is expanded to the user's home directory using `homedir.Expand`. No other environment variables are directly used within these files, but the expansion of `~` depends on the shell's environment setup.

## Launching Edge Cases:

The CLI can be launched with or without arguments. The `--version` flag will print version information and exit. If the `--config` flag is missing, the application will error out before executing any commands. Signals (SIGINT/SIGTERM) are handled gracefully via `WaitInterrupted`, allowing for clean shutdown.

## Code Relations & Unclear Places:

The relationship between these files is straightforward: `cobra.go` sets up the CLI structure and command execution flow, while `common.go` provides a utility function for handling signals during long-running operations. There are no apparent unclear places or dead code within this snippet. The package appears well-structured and focused on its core responsibilities.