# structs Package Summary

## Overview
The `structs` package provides two lightweight data structures that are used throughout the *insonmnia* system:

1. **ImagePush** – a wrapper around a gRPC stream that carries metadata for an image‑push task.
2. **NetworkSpec** – an enriched network specification that embeds the external `sonm.NetworkSpec` type and adds a unique identifier.

Both structures expose simple constructors, accessors, and helper functions that make it easy to create, validate, and pass around configuration data in gRPC calls.

---

## Project Package Structure

```
insonmnia/
└─ structs/
   ├─ image.go
   └─ network_spec.go
```

* `image.go` – defines the `ImagePush` type and its helpers.
* `network_spec.go` – defines the `NetworkSpec` type and batch‑creation utilities.

---

## Environment Variables, Flags & Cmdline Arguments

| Source | Description |
|--------|-------------|
| **Metadata keys** (`"deal"` / `"size"`) | Values extracted from a gRPC stream’s context in `NewImagePush`. |
| **Command line flag** – none defined explicitly; the package expects a `sonm.Worker_PushTaskServer` stream to be passed into `NewImagePush`. |
| **Environment variable** – not used directly, but the generated UUID for `NetworkSpec` is derived from `github.com/pborman/uuid`. |

---

## How the Application Can Be Launched

If this package is part of a command‑line or main application, it can be invoked in two typical ways:

1. **As a gRPC handler** – a server receives a `Worker_PushTaskServer` stream, calls `NewImagePush(stream)` to obtain an `*ImagePush`, and then uses the returned struct for further processing (e.g., writing image data to disk or forwarding it to another service).

2. **Batch creation of network specs** – a higher‑level routine can call `NewNetworkSpecs([]*sonm.NetworkSpec)` to convert a slice of external specs into enriched local structs, each with its own UUID (`NetID`). The resulting slice can then be passed to other components that need a list of network specifications.

---

## Detailed Code Summary

### 1. `image.go`

| Section | Purpose |
|---------|---------|
| **Imports** | `strconv` for string → int64 conversion; `github.com/sonm-io/core/proto` for the gRPC server interface; `google.golang.org/grpc/codes`, `metadata`, and `status` for handling gRPC metadata. |
| **Type `ImagePush`** | Holds a `Worker_PushTaskServer` stream, a deal identifier (`dealId`) and an image size (`imageSize`). |
| **Helper `requireHeader(md metadata.MD, name string)`** | Retrieves the last value of a named header from gRPC metadata. |
| **Helper `RequireHeaderInt64(md metadata.MD, name string)`** | Wraps `requireHeader` to parse the retrieved string into an int64. |
| **Constructor `NewImagePush(stream sonm.Worker_PushTaskServer) (*ImagePush, error)`** | Extracts `"deal"` and `"size"` from the stream’s context metadata, validates them, and returns a fully initialized struct. |
| **Accessors** | `DealId()` and `ImageSize()` simply expose the two fields for other components. |

### 2. `network_spec.go`

| Section | Purpose |
|---------|---------|
| **Imports** | `errors` & `strings` for error handling and string manipulation; `github.com/pborman/uuid` to generate a UUID; `github.com/sonm-io/core/proto` for the external `NetworkSpec`. |
| **Type `NetworkSpec`** | Embeds a pointer to `sonm.NetworkSpec` (so all its fields/methods are directly accessible) and adds a local identifier (`NetID`). |
| **Validation `validateNetworkSpec(id string, spec *sonm.NetworkSpec)`** | Ensures the embedded spec has a non‑empty type. |
| **Constructor `NewNetworkSpec(spec *sonm.NetworkSpec) (*NetworkSpec, error)`** | Generates a UUID (without hyphens), validates the supplied spec, and returns a new struct instance. |
| **Batch constructor `NewNetworkSpecs(specs []*sonm.NetworkSpec) ([]*NetworkSpec, error)`** | Accepts a slice of external specs, converts each into a local struct via `NewNetworkSpec`, and returns a slice of pointers to the newly created structs. |

---

## Relations Between Code Entities

| Entity | Relation |
|--------|----------|
| `ImagePush` ↔ `Worker_PushTaskServer` | The stream is stored directly in the struct, allowing downstream code to call gRPC methods on it. |
| `NewImagePush` ↔ `requireHeader` / `RequireHeaderInt64` | These helpers are used inside the constructor to pull metadata values from the incoming context. |
| `NetworkSpec` ↔ `sonm.NetworkSpec` | The embedded pointer gives direct access to all fields of the external type; `NetID` is an additional identifier that can be used for logging or database keys. |
| `NewNetworkSpecs` ↔ `NewNetworkSpec` | The batch constructor simply iterates over a slice and calls the single‑spec constructor, ensuring consistent validation logic. |

---

## Edge Cases & Potential Extensions

* **Missing metadata** – If `"deal"` or `"size"` are absent in the gRPC context, `requireHeader` will return an error; callers should handle this gracefully.
* **UUID generation** – The UUID is stripped of hyphens; if a different format is required, adjust the string replacement logic.
* **Error handling** – Currently errors propagate directly from constructors; adding logging or retry logic could improve robustness.

---