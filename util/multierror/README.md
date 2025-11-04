## Package: `multierror`

This package provides utilities for handling multiple errors in Go, leveraging the `github.com/hashicorp/go-multierror` library. It offers both standard and thread-safe multi-error implementations.

**Project Package Structure:**

```
util/
└── multierror/
    ├── error.go
```

**Configuration:**

*   No explicit configuration options are present in the provided code. The behavior is determined by the underlying `github.com/hashicorp/go-multierror` package and the custom `errorFormat` function.

**Usage:**

The package is designed to be used as a helper for aggregating errors, particularly in concurrent environments where thread safety is required. The `TSMultiError` type ensures that error appending is safe across multiple goroutines.

**Key Functions:**

*   `NewMultiError()`: Creates a new multi-error instance.
*   `NewTSMultiError()`: Creates a thread-safe multi-error instance.
*   `Append()`: Appends an error to the multi-error.
*   `AppendUnique()`: Appends an error only if it's not already present.
*   `ErrorOrNil()`: Returns the underlying error or nil if empty.

**Relations:**

The `TSMultiError` type wraps the standard `multierror.Error` to provide thread safety. The `errorFormat` function customizes the error output format.