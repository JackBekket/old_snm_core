## Package: datasize

This package provides custom data types (`ByteSize`, `BitRate`) for representing and manipulating data sizes with support for various units (Bytes, Bits, KB, MB, GB, etc.) in both binary and decimal formats. It includes parsing, formatting, and conversion functions, along with error handling for invalid inputs. The package is designed to handle human-readable size strings and perform accurate calculations.

**Configuration:**

*   No explicit environment variables, command-line arguments, or external configuration files are used. The package operates solely on input strings and internal constants.

**Edge Cases:**

*   The `UnmarshalText` methods for `BitRate` and `ByteSize` may exhibit unexpected behavior with empty strings or invalid formats due to the TODO comments in the code.
*   The parsing logic relies on string splitting and numeric conversions, which could lead to errors if the input strings are malformed.

**Project Structure:**

```
util/
└── datasize/
    ├── datasize.go
    └── datasize_test.go
```

**Relationships:**

*   `datasize.go` defines the core data structures and functions for size manipulation.
*   `datasize_test.go` provides unit tests to verify the correctness of the parsing, formatting, and conversion functions.

**Unclear Places/Dead Code:**

*   The TODO comments in `splitDimension` and `UnmarshalText` indicate potential issues with parsing behavior that may need further investigation.
*   The extensive use of constants for units could be simplified or refactored if necessary.