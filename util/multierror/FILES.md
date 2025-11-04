# util/multierror/error.go  
## Package: `multierror`  
  
**Imports:**  
  
*   `strings`  
*   `sync`  
*   `github.com/hashicorp/go-multierror`  
  
**External Data/Input Sources:**  
  
*   Relies on the `github.com/hashicorp/go-multierror` package for core multi-error functionality.  
*   Uses standard library `strings` for joining error messages.  
*   Uses `sync` for thread-safe operations in `TSMultiError`.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Summary of Code Parts:**  
  
### MultiError Creation  
  
The `NewMultiError()` function creates a new `multierror.Error` instance using a predefined `errorFormat` function. `NewTSMultiError()` creates a thread-safe wrapper around `multierror.Error`.  
  
### Error Appending  
  
The `Append()` function is a direct wrapper around `github.com/hashicorp/go-multierror.Append()`, providing a convenient way to add errors to a multi-error. `AppendUnique()` prevents duplicate errors based on their string representation.  
  
### Error Formatting  
  
The `errorFormat()` function takes a slice of errors and joins their string representations with commas, providing a custom format for multi-error output.  
  
### Thread-Safe MultiError (`TSMultiError`)  
  
The `TSMultiError` type wraps `multierror.Error` with a `sync.Mutex` to ensure thread-safe appending of errors. The `Append()` method locks the mutex before appending errors and unlocks it afterward. `ErrorOrNil()` returns the underlying error or nil if empty.  
  
