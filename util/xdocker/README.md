```markdown
# xdocker Package Summary

**Package Name:** `xdocker`

This package provides utilities for parsing, manipulating, and decoding Docker image references and pull streams. It wraps existing Docker reference parsing logic with custom types and methods while also handling JSON-encoded responses from the Docker daemon during image pulls. The core functionality revolves around validating and extracting information from Docker image names, tags, and digests.

**Configuration:**

*   No explicit environment variables or command-line arguments are present in this code summary.
*   The `DecodeImagePull` function relies on an `io.Reader` as input, which can be configured externally (e.g., standard input, file stream, network connection).
*   Integration tests use a running Docker daemon; ensure the Docker environment is set up correctly before execution.

**Project Package Structure:**

```
util/xdocker/
├── reference.go       # Reference parsing and manipulation logic
├── reference_test.go  # Unit tests for reference functionality
├── xdocker.go          # JSON decoding from Docker pull streams
└── xdocker_test.go     # Tests for stream decoding and integration with Docker daemon
```

**Code Relations:**

*   `reference.go` defines the `Reference` type, providing a wrapper around Docker's native reference parsing. It includes methods for extracting name, tag, and digest information.
*   `xdocker.go` focuses on handling JSON responses from Docker during image pulls. The `DecodeImagePull` function reads line by line, parses each as JSON, and checks for error conditions.
*   The test files (`*_test.go`) verify the correctness of these functions through unit tests (mock data) and integration tests (actual Docker daemon interaction).

**Edge Cases:**

*   If `NewReference` receives an invalid image reference string, it returns an error instead of panicking.
*   `DecodeImagePull` handles malformed JSON responses by iterating until an error is encountered or the input stream ends. It specifically checks for non-empty "Error" fields in the decoded responses.

<end_of_output>
```