## insomniia/structs

This package defines data structures and utility functions related to image pushing and network specifications within a SONM-based system. It heavily relies on gRPC metadata for validation and configuration.

**Project Structure:**

```
insonmnia/
├── structs/
│   ├── image.go
│   └── network_spec.go
```

**Configuration & Environment Variables:**

*   **gRPC Metadata Headers (image.go):** The `deal` and `size` headers in incoming gRPC metadata are critical for the `ImagePush` struct's functionality. Missing or invalid values will cause errors. No environment variables directly configure this behavior, but external services providing these headers must be properly configured.
*   **External Dependencies (network_spec.go):** The package depends on `github.com/pborman/uuid` for UUID generation and `github.com/sonm-io/core/proto` for the base `NetworkSpec` structure. These dependencies need to be correctly installed and configured in the build environment.

**Edge Cases & Launch Arguments:**

*   The package is a library, not an executable. It's intended to be used within other applications (e.g., gRPC servers).
*   Invalid or missing gRPC metadata headers will cause errors during `ImagePush` creation. The application using this struct must handle these errors gracefully.
*   If the `Type` field in a `sonm.NetworkSpec` is empty, `NewNetworkSpec` will return an error.

**Code Relations & Unclear Places:**

*   The `image.go` file focuses on validating gRPC metadata before processing image data. The reliance on specific header names ("deal", "size") makes it tightly coupled to the upstream service definition.
*   The `network_spec.go` file extends the external `sonm.NetworkSpec` struct by adding a UUID-based `NetID`. The purpose of this ID is unclear without further context, but it likely serves as a unique identifier for network specifications within the system.
*   The validation function in `network_spec.go` only checks if the `Type` field is empty and does not use the provided ID string. This seems redundant or incomplete.