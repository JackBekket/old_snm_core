## Package: `structs`

This package defines custom structs and functions for handling image pushing tasks and network specifications within the SONM framework. It relies heavily on gRPC metadata and the `sonm-io/core/proto` package.

**Project Package Structure:**

*   `insonmnia/structs/image.go`
*   `insonmnia/structs/network_spec.go`

**Configuration:**

*   **Environment Variables:** None explicitly used in the provided code.
*   **Cmdline Arguments:** None.
*   **Files:** The code relies on gRPC metadata passed through the `sonm.Worker_PushTaskServer` interface. Specifically, the `deal` and `size` headers are required for `ImagePush`.
*   **Flags:** None.

**Edge Cases:**

*   `NewImagePush` fails if the `deal` or `size` headers are missing from the gRPC metadata.
*   `RequireHeaderInt64` fails if the `size` header cannot be parsed as an int64.
*   `NewNetworkSpec` fails if the `GetType()` field of the input `sonm.NetworkSpec` is empty.
*   `NewNetworkSpecs` fails if any of the individual `NewNetworkSpec` calls fail.

**Code Relations:**

*   `ImagePush` is designed to handle image pushing tasks, extracting metadata from gRPC streams.
*   `NetworkSpec` wraps the core `sonm.NetworkSpec` type, adding a unique identifier (`NetID`).
*   `validateNetworkSpec` ensures the integrity of the input `sonm.NetworkSpec` before creating a `NetworkSpec` instance.
*   `NewNetworkSpec` and `NewNetworkSpecs` provide factory functions for creating `NetworkSpec` instances, either individually or in bulk.