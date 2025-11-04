# Config Package Summary

The **config** package (located under `util/config`) provides a small, reusable helper for loading, transforming and saving YAML configuration files.  
It also contains a lightweight “tagger” that can generate snake‑case struct tags from Go structs.

---

## Files & Paths

```
util/
└─ config/
   ├─ config.go          # LoadWith – round‑trip load/modify/save
   ├─ retag.go           # SnakeCaseTagger, MakeTag, toSnakeCase, SnakeToLower
   └─ retag_test.go      # unit test for toSnakeCase
```

---

## Core Functionality

| File | Key Elements |
|------|---------------|
| **config.go** | `LoadWith(dst interface{}, path string, fn func(map[interface{}]interface{})) error` – reads a YAML file into a generic map, lets the caller mutate it via a callback, then writes it back and finally unmarshals into the supplied destination. |
| **retag.go** | * `SnakeCaseTagger` – alias for `string`. <br> * `MakeTag(fieldIndex int, t reflect.Type) reflect.StructTag` – builds a struct tag in snake‑case form for a field of type `t`. <br> * `toSnakeCase(s string) string` – converts any string into lower‑cased snake‑case. <br> * `isDelimiter(r rune) bool` – helper used by `toSnakeCase`. <br> * `SnakeToLower(m map[interface{}]interface{})` – recursively walks a map and normalizes all keys to lower‑case snake form. |
| **retag_test.go** | Unit test that verifies `toSnakeCase` against a wide range of inputs. |

---

## Environment Variables / Flags

* **Path to config file** – passed as the second argument to `LoadWith`.  
  Example: `config.LoadWith(&cfg, "config.yaml", func(m map[interface{}]interface{}) { … })`.

* **Destination variable** – any Go value that implements `yaml.Unmarshal` (e.g., a struct).  

No explicit command‑line flags are defined in this package; it is intended to be used programmatically.

---

## How the Application Can Be Launched

1. **As a library**  
   Import `"util/config"` and call `LoadWith` from any other Go file:

   ```go
   var cfg MyConfigStruct
   err := config.LoadWith(&cfg, "config.yaml", func(m map[interface{}]interface{}) {
       // optional modifications before saving
   })
   ```

2. **As a CLI helper**  
   If the project contains a `main` package that needs to read/write configuration files, it can invoke `LoadWith` directly or wrap it in its own command‑line interface.

---

## Summary of Logic

* `config.go` implements a generic load–modify–save cycle for YAML files.  
  It reads the file into memory, unmarshals into an intermediate map (`ty`), lets the caller mutate that map via a callback, marshals it back to bytes, and finally writes the result into the supplied destination interface.

* `retag.go` supplies helper types and functions for generating snake‑case struct tags.  
  The `SnakeCaseTagger.MakeTag` method builds a tag string like ``key:"value"`` using the `toSnakeCase` routine.  
  `SnakeToLower` walks nested maps recursively, converting all keys to lower‑cased snake form.

* `retag_test.go` ensures that `toSnakeCase` behaves correctly across many edge cases (empty strings, camel case, spaces, dashes, underscores, etc.).

Overall, the package offers a compact yet flexible way to handle YAML configuration files and to generate struct tags for Go structs.