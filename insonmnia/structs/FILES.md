# insonmnia/structs/image.go  
**Package / Component**    
`structs`  
  
---  
  
### Imports  
```go  
import (  
	"strconv"  
  
	"github.com/sonm-io/core/proto"  
	"google.golang.org/grpc/codes"  
	"google.golang.org/grpc/metadata"  
	"google.golang.org/grpc/status"  
)  
```  
* `strconv` – for parsing string to int64.    
* `github.com/sonm-io/core/proto` – contains the gRPC server interface used (`Worker_PushTaskServer`).    
* `google.golang.org/grpc/codes`, `metadata`, `status` – gRPC utilities for status handling and metadata extraction.  
  
---  
  
### External Data / Input Sources  
| Source | Description |  
|--------|-------------|  
| `stream.Context()` | Incoming context from a gRPC call, used to obtain metadata. |  
| Metadata key `"deal"` | Header value parsed as string (used by `requireHeader`). |  
| Metadata key `"size"` | Header value parsed as int64 (used by `RequireHeaderInt64`). |  
  
---  
  
### TODOs  
No explicit TODO comments are present in this file.  
  
---  
  
## Summary of Major Code Parts  
  
#### ### Struct Definition  
```go  
type ImagePush struct {  
	sonm.Worker_PushTaskServer  
	dealId    string  
	imageSize int64  
}  
```  
* Holds a gRPC server stream (`Worker_PushTaskServer`) and two fields: `dealId` (string) and `imageSize` (int64).    
* Provides a lightweight container for data needed during an image‑push task.  
  
#### ### Helper Functions  
1. **`requireHeader(md metadata.MD, name string)`** – retrieves the last value of a named header from gRPC metadata; returns it as a string or an error if missing.  
2. **`RequireHeaderInt64(md metadata.MD, name string)`** – wraps `requireHeader`, converting the retrieved string into an int64 using `strconv.ParseInt`.  
  
#### ### Constructor  
```go  
func NewImagePush(stream sonm.Worker_PushTaskServer) (*ImagePush, error)  
```  
* Extracts incoming metadata from the provided stream context.    
* Reads `"deal"` and `"size"` headers via the helper functions.    
* Returns a fully initialized `ImagePush` instance or an error if any step fails.  
  
#### ### Accessor Methods  
```go  
func (p *ImagePush) DealId() string  
func (p *ImagePush) ImageSize() int64  
```  
* Simple getters for the two fields, enabling other components to read the deal ID and image size from an `ImagePush` instance.  
  
---  
  
These sections together provide a concise wrapper around gRPC metadata handling for pushing images in the system.  
  
# insonmnia/structs/network_spec.go  
**Package / Component**    
`structs`  
  
---  
  
### Imports  
```go  
import (  
	"errors"  
	"strings"  
  
	"github.com/pborman/uuid"  
	"github.com/sonm-io/core/proto"  
)  
```  
* `errors` – standard Go package for error handling.    
* `strings` – standard Go package for string manipulation.    
* `github.com/pborman/uuid` – third‑party UUID generator used to create unique identifiers.    
* `github.com/sonm-io/core/proto` – external data source; contains the definition of `sonm.NetworkSpec`.  
  
---  
  
### External Data / Input Sources  
| Source | Description |  
|--------|-------------|  
| `github.com/sonm-io/core/proto` | Provides the base type `sonm.NetworkSpec`. The struct defined in this file embeds that type and adds a local identifier (`NetID`). |  
  
---  
  
### TODOs  
No explicit TODO comments were found in the code.  
  
---  
  
## Summary of Major Code Parts  
  
### 1. Type Definition – `NetworkSpec`  
```go  
type NetworkSpec struct {  
	*sonm.NetworkSpec  
	NetID string  
}  
```  
* Embeds a pointer to `sonm.NetworkSpec` so that all fields and methods from the external type are available directly on `structs.NetworkSpec`.    
* Adds an additional field `NetID`, which holds a unique identifier for each network specification.  
  
### 2. Validation – `validateNetworkSpec`  
```go  
func validateNetworkSpec(id string, spec *sonm.NetworkSpec) error {  
	if len(spec.GetType()) == 0 {  
		return errors.New("network type is required in network spec")  
	}  
	return nil  
}  
```  
* Checks that the embedded `sonm.NetworkSpec` contains a non‑empty type.    
* Returns an error if validation fails; otherwise returns `nil`.  
  
### 3. Constructor – `NewNetworkSpec`  
```go  
func NewNetworkSpec(spec *sonm.NetworkSpec) (*NetworkSpec, error) {  
	id := strings.Replace(uuid.New(), "-", "", -1)  
	err := validateNetworkSpec(id, spec)  
	if err != nil {  
		return nil, err  
	}  
	return &NetworkSpec{spec, id}, nil  
}  
```  
* Generates a UUID string (removing hyphens), validates the supplied `sonm.NetworkSpec`, and returns a new `structs.NetworkSpec` instance with the generated ID.  
  
### 4. Batch Constructor – `NewNetworkSpecs`  
```go  
func NewNetworkSpecs(specs []*sonm.NetworkSpec) ([]*NetworkSpec, error) {  
	result := make([]*NetworkSpec, 0, len(specs))  
	for _, s := range specs {  
		spec, err := NewNetworkSpec(s)  
		if err != nil {  
			return nil, err  
		}  
		result = append(result, spec)  
	}  
	return result, nil  
}  
```  
* Accepts a slice of `sonm.NetworkSpec` pointers and converts each into the local `structs.NetworkSpec`.    
* Returns a slice of pointers to the newly created specs or an error if any conversion fails.  
  
---  
  
**<end_of_output>**  
  
