# util/xdocker/reference.go  
**Package / Component**    
`xdocker`  
  
---  
  
### Imports    
```go  
import (  
	"github.com/docker/distribution/reference"  
	"github.com/opencontainers/go-digest"  
	"github.com/pkg/errors"  
)  
```  
* `reference` – Docker distribution reference handling (parsing, tagging, digesting).    
* `digest` – OpenContainers digest type.    
* `errors` – Error handling for the package.  
  
---  
  
### External data / input sources    
The code works with a string representation of a Docker image reference (`source`).    
All methods ultimately rely on the underlying `reference.Reference` interface from the Docker distribution library, which can be parsed, marshalled/unmarshalled, and extended with tags or digests.  
  
---  
  
### TODOs    
No explicit TODO comments are present in this file.  
  
---  
  
## Summary of major code parts  
  
| Section | Purpose | Key points |  
|---------|---------|------------|  
| **Type definition** | `Reference` struct wraps a Docker reference. | Embeds `reference.Reference`; allows direct access to all methods of the underlying interface. |  
| **NewReference** | Factory that creates a `Reference` from a string source. | Calls `Parse`, returns an error if parsing fails. |  
| **Parse** | Parses any reference string into the embedded `reference.Reference`. | Uses `reference.ParseAnyReference`; assigns to the struct field. |  
| **UnmarshalText / MarshalText** | Implements text marshaling for the type. | `UnmarshalText` delegates to `Parse`; `MarshalText` returns the string representation as bytes. |  
| **Named** | Retrieves the underlying named reference if available. | Type‑asserts to `reference.Named`, returning nil when not applicable. |  
| **WithTag** | Adds a tag to the current reference and returns a new `Reference`. | Uses `reference.TrimNamed` and `reference.WithTag`; error handling included. |  
| **HasDigest / Digest** | Checks for digest presence and retrieves it. | `HasDigest` checks interface assertion; `Digest` returns the digest value or empty string if not present. |  
| **WithDigest** | Adds a digest to the reference, returning a new `Reference`. | Similar pattern to `WithTag`; uses `reference.WithDigest`. |  
| **HasName / Name** | Checks for and retrieves the name part of the reference. | Uses interface assertion to `reference.Named` and returns the name string. |  
  
---  
  
All methods are straightforward wrappers around the Docker distribution library, providing a convenient API for creating, inspecting, and extending Docker image references within the `xdocker` package.  
  
# util/xdocker/reference_test.go  
# xdocker Package – Reference Marshal/Unmarshal Test  
  
## Imports  
```go  
import (  
	"encoding/json"  
	"testing"  
  
	"github.com/stretchr/testify/require"  
)  
```  
* `encoding/json` – standard library for JSON marshaling/unmarshaling.  
* `testing` – Go testing framework used to run the unit test.  
* `github.com/stretchr/testify/require` – assertion helpers from Testify.  
  
## External Data / Input Sources  
| Variable | Value | Purpose |  
|----------|-------|---------|  
| `refStr` | `"httpd:latest"` | Initial reference string passed to `NewReference`. |  
| `data`   | JSON bytes of a `Reference` instance | Result of marshaling the reference. |  
  
The test also uses the output of `json.Unmarshal` into a new `Reference` pointer (`ref2`) and compares its string representation.  
  
## TODOs  
No explicit TODO comments are present in this file; all functionality is exercised by the single test function.  
  
## Summary of Major Code Parts  
  
### Test Function: `TestReferenceMarshalUnmarshal`  
* **Purpose** – Verify that a `Reference` can be created from a string, marshaled to JSON, and unmarshaled back while preserving its canonical form.  
* **Steps**  
  1. Create a reference from the literal `"httpd:latest"` using `NewReference`.  
  2. Assert no error occurs during creation (`require.NoError(t, err)`).  
  3. Marshal the reference into JSON bytes; assert success and that the resulting string equals `"docker.io/library/httpd:latest"`.  
  4. Unmarshal those bytes back into a new `Reference` pointer.  
  5. Assert no error on unmarshaling and that the string representation of the new reference matches the expected canonical form.  
  
The test confirms both the correctness of the `NewReference` constructor and the JSON (un)marshaling logic for the `Reference` type, ensuring round‑trip fidelity.  
  
# util/xdocker/xdocker.go  
**Package / Component Name**    
`xdocker`  
  
---  
  
### Imports  
```go  
import (  
	"bufio"  
	"bytes"  
	"encoding/json"  
	"fmt"  
	"io"  
)  
```  
* `bufio`: buffered I/O reader for reading from an `io.Reader`.  
* `bytes`: byte slice utilities, used to create a new reader.  
* `encoding/json`: JSON decoding of Docker responses.  
* `fmt`: formatting and error handling.  
* `io`: generic I/O interface.  
  
---  
  
#### External Data / Input Sources  
| Function | Input Source | Description |  
|----------|--------------|-------------|  
| `DecodeImagePull` | `io.Reader` (e.g., a network connection or file) | Reads Docker pull responses line‑by‑line and decodes each JSON object. |  
  
---  
  
#### TODO Comments  
No explicit `TODO:` comments are present in the current code.  
  
---  
  
## Summary of Major Code Parts  
  
#### 1. `spoolResponseProtocol`  
```go  
type spoolResponseProtocol struct {  
	Error  string `json:"error"`  
	Status string `json:"status"`  
}  
```  
*Defines the structure expected from Docker’s pull response: an error message and a status field, both decoded from JSON.*  
  
#### 2. `DecodeImagePull`  
```go  
func DecodeImagePull(r io.Reader) error { … }  
```  
*Creates a buffered reader (`bufio.NewReader`) to read the incoming stream line by line until EOF or an empty line is encountered.*  
* For each line:  
  * Trims the trailing newline.  
  * Calls `decodePullLine` to parse the JSON into a `spoolResponseProtocol`.  
* Returns any error that occurs during reading or decoding.  
  
#### 3. `decodePullLine`  
```go  
func decodePullLine(line []byte) error { … }  
```  
*Wraps the line bytes in a `bytes.Reader`, then uses `json.NewDecoder` to unmarshal into a `spoolResponseProtocol`.*  
* The loop inside this function appears intended to handle multiple JSON objects per line, but currently it only processes one and returns on success.*  
  
---  
  
**<end_of_output>**  
  
# util/xdocker/xdocker_test.go  
**Package / Component**    
`xdocker`  
  
---  
  
### Imports  
```go  
import (  
	"bytes"  
	"context"  
	"fmt"  
	"log"  
	"testing"  
  
	"github.com/docker/docker/api/types"  
	"github.com/docker/docker/client"  
	"github.com/stretchr/testify/assert"  
)  
```  
The file pulls in the standard library packages for I/O, context handling and testing, plus Docker client types and a test assertion helper.  
  
---  
  
### External Data / Input Sources  
* **Fixtures** – an inline slice of anonymous structs that provide:  
  * `name` – a descriptive label for each case.  
  * `body` – raw JSON payloads to be fed into the decoder.  
  * `err` – expected error value (currently all are `nil`, except two cases that expect a non‑nil error).  
* **Docker client** – created via `client.NewEnvClient()` and used in `TestImagePull`.  
  
---  
  
### TODO Comments  
No explicit `TODO:` markers were found in the file.    
(If future work is needed, add a section for pending tasks.)  
  
---  
  
## Summary of Major Code Parts  
  
#### 1. Test Fixture Construction (`TestImagePullFromMock`)  
* Builds eight distinct test cases covering:  
  * Single line JSON.  
  * Multiple lines with and without trailing newline.  
  * Flat concatenated JSON.  
  * Mixed combinations of the above.  
* Each case is executed in a loop that calls `DecodeImagePull` on a `bytes.Reader` created from the fixture body, then verifies the returned error matches the expected value.  
  
#### 2. Docker Image Pull Test (`TestImagePull`)  
* Instantiates a Docker client with environment defaults.  
* Calls `dockclient.ImagePull` to pull an image named `"alpine:latest"` using empty options.  
* Defers closing of the returned reader and immediately decodes it via `DecodeImagePull`.  
* Errors are surfaced through the test harness (`t.Fatal`) if any step fails.  
  
#### 3. Common Functionality  
Both tests rely on a shared helper, `DecodeImagePull`, which is assumed to exist elsewhere in the package.    
The first test validates that this decoder correctly interprets various JSON payload shapes; the second verifies it works against an actual Docker pull stream.  
  
---  
  
**End of output**  
  
