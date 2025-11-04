## Package: `secterm`

This package implements a command-line tool (`secterm`) for establishing and running a remote PTY (pseudo-terminal) connection. It leverages the `secsh` library for secure shell-like functionality.

**Project Structure:**

```
cmd/secterm/
├── main.go
```

**Configuration:**

*   **`ADDR` (cmdline argument):** Required. Specifies the target address for the remote PTY connection.
*   **`--config` (cmdline flag):** Optional. Specifies the path to a YAML configuration file. Defaults to `etc/secterm.yaml`.

**Environment Variables:**

None explicitly used in the provided code.

**Edge Cases (Launch):**

*   **Missing `ADDR`:** The program will exit with an error if the target address is not provided as a command-line argument.
*   **Invalid Configuration File:** If the `--config` flag is used with an invalid path or a malformed YAML file, the program will exit with an error.
*   **Remote PTY Failure:** If the remote PTY connection fails (e.g., due to network issues or authentication errors), the program will exit with an error.

**Code Logic:**

1.  **CLI Definition:** Uses `cobra` to define the `secterm` command, which takes the target address (`ADDR`) as a required argument and supports the `--config` flag for specifying a configuration file.
2.  **Configuration Loading:** Loads configuration from the YAML file (if provided) using `secshc.NewRPTYConfig`.
3.  **Remote PTY Creation:** Creates a remote PTY object using `secshc.NewRemotePTY` based on the loaded configuration.
4.  **PTY Execution:** Executes the remote PTY using `tty.Run` with the target address.
5.  **Error Handling:** Includes robust error handling at each step, printing error messages to the console and exiting with a non-zero exit code.

**Dependencies:**

*   `context`
*   `fmt`
*   `os`
*   `github.com/sonm-io/core/insonmnia/auth`
*   `github.com/sonm-io/core/secsh/secshc`
*   `github.com/spf13/cobra`