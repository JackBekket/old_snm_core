# xdocker

The **xdocker** package supplies a small set of helpers for working with Docker image references and for decoding the JSON stream that is returned when pulling an image.  
It wraps the `github.com/docker/distribution/reference` API to make it easier to create, inspect and extend references, and provides a lightweight reader that turns a raw pull response into Go structs.

---

## Short summary

* **Reference handling** – A thin wrapper around Docker’s reference type with helpers for parsing, marshaling/unmarshaling, tagging and digesting.  
* **Pull‑response decoding** – Reads an `io.Reader` line by line, unmarshals each JSON object into a `spoolResponseProtocol`, and returns any error that occurs.

---

## Files & paths

```
util/xdocker/
├── reference.go
├── reference_test.go
├── xdocker.go
└── xdocker_test.go
```

### `reference.go`

* Defines the `Reference` struct and all public methods (`NewReference`, `Parse`, `UnmarshalText`, `MarshalText`, `Named`, `WithTag`, `HasDigest`, `Digest`, `WithDigest`, `HasName`, `Name`).  
* Uses the Docker distribution reference library to parse a string, add tags/digests, and expose the underlying named reference.

### `reference_test.go`

* One test (`TestReferenceMarshalUnmarshal`) that verifies round‑trip correctness of the wrapper: create → marshal → unmarshal → compare canonical string.

### `xdocker.go`

* Declares `spoolResponseProtocol` struct with fields `Error` and `Status`.  
* Implements `DecodeImagePull(r io.Reader) error`, which reads a pull stream line by line, trims newlines, calls `decodePullLine`, and returns any error.  
* Helper `decodePullLine(line []byte) error` wraps the raw bytes in a `bytes.Reader`, decodes into `spoolResponseProtocol`, and currently processes only one JSON object per call.

### `xdocker_test.go`

* Two tests:  
  * `TestImagePullFromMock` – feeds various mock payloads to `DecodeImagePull` and checks that no error is returned.  
  * `TestImagePull` – pulls an image via the Docker client, defers closing of the reader, and immediately decodes it with `DecodeImagePull`.

---

## Environment variables / flags / command‑line arguments

| Source | Variable / flag | Purpose |
|--------|-----------------|---------|
| None explicitly defined in this package.  The tests rely on the standard Go test harness (`go test ./...`). |

If you want to run the pull decoder manually, you can invoke:

```bash
go test ./util/xdocker -run TestImagePullFromMock
```

or use the Docker client directly from code:

```go
client, _ := client.NewEnvClient()
reader, err := client.ImagePull(context.Background(), "alpine:latest", nil)
defer reader.Close()
err = xdocker.DecodeImagePull(reader)
```

---

## Edge cases of launching

* **Unit tests** – `go test ./util/xdocker` will run both reference and pull‑decoder tests.  
* **Integration** – The package can be imported into a larger Docker‑related tool; the exported `Reference` type and `DecodeImagePull` function are the primary entry points.

---

## Relations between code entities

| Entity | Relation |
|--------|----------|
| `Reference` struct | Embeds `reference.Reference`; all methods operate on that embedded value. |
| `NewReference`, `Parse`, `UnmarshalText`, `MarshalText` | Provide construction and (un)marshaling logic for the wrapper. |
| `WithTag`, `WithDigest` | Return new `Reference` values with added tag/digest; they internally call Docker distribution helpers (`TrimNamed`, `WithTag`, `WithDigest`). |
| `DecodeImagePull` ↔ `decodePullLine` | The former reads a stream and delegates each line to the latter. |
| Tests ↔ implementation | `reference_test.go` validates the wrapper; `xdocker_test.go` validates the decoder against both mock data and an actual Docker pull response. |

No dead code was detected; all functions are exercised by the tests.

---

**<end_of_output>**