# insonmnia/structs/image.go  
## Package: `structs`  
  
**Imports:**  
  
*   `strconv`: For string to integer conversion.  
*   `github.com/sonm-io/core/proto`: SONM core proto definitions.  
*   `google.golang.org/grpc/codes`: gRPC error codes.  
*   `google.golang.org/grpc/metadata`: gRPC metadata handling.  
*   `google.golang.org/grpc/status`: gRPC status error creation.  
  
**External Data/Input Sources:**  
  
*   gRPC metadata (`metadata.MD`) from incoming context. Specifically, the `deal` and `size` headers are required.  
*   The `sonm.Worker_PushTaskServer` interface, which provides the gRPC stream context.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
---  
  
### `ImagePush` Struct  
  
The `ImagePush` struct embeds `sonm.Worker_PushTaskServer` and stores the `dealId` (string) and `imageSize` (int64). It's designed to handle image pushing tasks within the SONM framework.  
  
### Header Extraction Functions  
  
*   `requireHeader`: Extracts a string value from gRPC metadata by header name. Returns an error if the header is missing.  
*   `RequireHeaderInt64`: Extracts a string value from gRPC metadata, parses it as an int64, and returns an error if parsing fails or the header is missing.  
  
### `NewImagePush` Function  
  
This function constructs an `ImagePush` instance from a `sonm.Worker_PushTaskServer` stream. It extracts the `deal` and `size` headers from the incoming gRPC metadata, converts `size` to an int64, and initializes the `ImagePush` struct.  Returns an error if metadata is missing or invalid.  
  
### Accessor Methods  
  
*   `DealId()`: Returns the stored `dealId`.  
*   `ImageSize()`: Returns the stored `imageSize`.  
  
# insonmnia/structs/network_spec.go  
## Package: `structs`  
  
**Imports:**  
  
*   `errors`: For error handling.  
*   `strings`: For string manipulation (specifically, removing hyphens from UUIDs).  
*   `github.com/pborman/uuid`: For generating UUIDs.  
*   `github.com/sonm-io/core/proto`: For using `sonm.NetworkSpec` type.  
  
**External Data/Input Sources:**  
  
*   `sonm.NetworkSpec`: This struct is used as input to create `NetworkSpec` instances. The validity of this input is checked by `validateNetworkSpec`.  
*   UUID generation: The code relies on the `github.com/pborman/uuid` package to generate unique identifiers.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
**Summary of Code Parts:**  
  
### `NetworkSpec` Struct  
  
Defines a custom struct `NetworkSpec` that embeds `sonm.NetworkSpec` and adds a `NetID` field. This struct appears to be a wrapper around the core `sonm.NetworkSpec` type, adding an identifier.  
  
### `validateNetworkSpec` Function  
  
Validates a `sonm.NetworkSpec` instance, ensuring that the `GetType()` field is not empty. Returns an error if the type is missing.  
  
### `NewNetworkSpec` Function  
  
Creates a new `NetworkSpec` instance from a `sonm.NetworkSpec`. It generates a UUID (without hyphens) and validates the input spec using `validateNetworkSpec`. Returns the new `NetworkSpec` or an error if validation fails.  
  
### `NewNetworkSpecs` Function  
  
Creates a slice of `NetworkSpec` instances from a slice of `sonm.NetworkSpec` instances. It iterates through the input slice, creating each `NetworkSpec` using `NewNetworkSpec`. Returns the resulting slice or an error if any individual creation fails.  
  
