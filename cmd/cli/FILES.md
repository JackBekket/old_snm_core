# cmd/cli/main.go  
## Package: `main`  
  
**Imports:**  
  
*   `os`: Standard library package for operating system functionalities (specifically, `os.Exit`).  
*   `github.com/sonm-io/core/cmd/cli/commands`: Contains the root command and execution logic for the CLI.  
*   `github.com/sonm-io/core/insonmnia/version`: Provides the application version information.  
  
**External Data/Input Sources:**  
  
*   The program relies on command-line arguments passed to the CLI, handled by the `github.com/sonm-io/core/cmd/cli/commands` package.  
*   The application version is sourced from the `github.com/sonm-io/core/insonmnia/version` package.  
  
**TODOs:**  
  
*   None found in this specific file.  
  
**Summary of Major Code Parts:**  
  
*   **Main Function:** The `main` function initializes the root command using `commands.Root` and executes it. Error handling is implemented to display errors using `commands.ShowError` and exit the program with a non-zero status code if an error occurs. The version information is passed to the root command from the `version` package.  
  
