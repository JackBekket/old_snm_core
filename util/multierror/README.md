# multierror

## Overview
`util/multierror/error.go` implements a small wrapper around Hashicorp’s `go-multierror` package, adding convenient constructors and a thread‑safe append helper.  
The file defines:

| Entity | Purpose |
|--------|---------|
| `NewMultiError()` | Creates a new `multierror.Error` with a custom formatting function. |
| `NewTSMultiError()` | Builds a thread‑safe wrapper (`TSMultiError`) around the error list. |
| `Append(err error, errs ...error)` | Thin helper that forwards to `multierror.Append`. |
| `AppendUnique(err *multierror.Error, other error)` | Adds an error only if it is not already present in the list. |
| `errorFormat(errs []error) string` | Formats a slice of errors into a comma‑separated string used by `NewMultiError`. |
| `TSMultiError` struct | Holds a mutex and an inner pointer to a `multierror.Error`. |
| `(m *TSMultiError).Append(errs ...error)` | Thread‑safe append method. |
| `(m *TSMultiError).ErrorOrNil()` | Returns the underlying error value. |

The package imports only standard library packages (`strings`, `sync`) and the external dependency `github.com/hashicorp/go-multierror`.

## Project structure
```
util/
└─ multierror/
   └─ error.go
```

## Configuration options
* **Environment variables** – none defined in this file.  
* **Flags / command‑line arguments** – none; the package is intended to be imported by other code.  
* **Files & paths for configuration** – only `util/multierror/error.go` exists, so any configuration must come from the exported functions above.

## Edge cases for launching
The file itself does not provide a CLI entry point, but it can be used in two ways:

1. **Direct use**:  
   ```go
   err := multierror.NewMultiError()
   err.Append(err1, err2)
   ```
2. **Thread‑safe wrapper**:  
   ```go
   tsErr := multierror.NewTSMultiError()
   tsErr.Append(err1, err2, err3)
   ```

Both patterns are thread‑safe; the `TSMultiError` type uses a mutex to guard concurrent appends.

## Relations between code entities
* `NewMultiError()` creates an instance of `multierror.Error`, setting its `ErrorFormat` field to the local `errorFormat` function.  
* `AppendUnique()` iterates over the supplied errors, checks for duplicates by comparing their string representations, and forwards them to `multierror.Append`.  
* The thread‑safe wrapper (`TSMultiError`) embeds a mutex and an inner pointer; its methods lock/unlock around calls to `multierror.Append`, ensuring safe concurrent access.  

No dead code or missing references are apparent in this file.