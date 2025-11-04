# Package `datasize`

The **util/datasize** directory contains a small Go library for handling data‑size values in bits, bytes and rates.  
It defines two public types (`ByteSize`, `BitRate`) that wrap an underlying numeric type (`bitSize`).  
Both types expose parsing, formatting and conversion helpers that are exercised by the accompanying test file.

---

## Project structure

```
util/
└─ datasize/
   ├─ datasize.go
   └─ datasize_test.go
```

* `datasize.go` – implementation of the library.  
* `datasize_test.go` – unit tests that validate parsing, marshaling and human‑readable output.

---

## Imports

| File | Packages |
|------|----------|
| `datasize.go` | `errors`, `fmt`, `strconv`, `strings` |
| `datasize_test.go` | `testing`, `github.com/stretchr/testify/require` |

The test file uses the *testify* package for concise assertions.

---

## Types & constants

```go
type bitSize uint64          // base numeric type
type ByteSize struct{bitSize} // represents a size in bytes
type BitRate struct{bitSize}  // represents a rate (bits per second)
```

| Constant | Meaning |
|----------|---------|
| `B`, `KiB`, `MiB`, … | binary prefixes for byte sizes |
| `KB`, `MB`, `GB`, … | decimal prefixes for byte sizes |
| `bit`, `Kibit`, `Mibit`, … | binary prefixes for bit rates |
| `kbit`, `Mbit`, … | numeric values of the above prefixes |
| `bitDimFlag`, `byteDimFlag`, `decimalDimFlag`, `binaryDimFlag` | flag mask that indicates whether a spelling is byte/bit and decimal/binary |
| `fnUnmarshalText` | key used in unmarshaling logic (e.g. “bit” or “MB”) |
| `maxUint64`, `cutoff` | limits for numeric conversion |

---

## Core helpers

* **`dimensionCount(dimension bitSize) float64`** – calculates how many of a given dimension fit into the current value.
* **`splitDimension(text []byte) (dimension string, size interface{}, err error)`** – splits an input slice into a numeric part and a suffix.  
  *TODO*: comment suggests handling empty strings gracefully.
* **`getPossibleDimensions(flags uint8) []string`** – returns all spellings that match a flag mask; used for error messages.

---

## Methods on `bitSize`

| Method | Purpose |
|--------|---------|
| `Bits()` / `Bytes()` | raw numeric value in bits or bytes |
| `KBits()`, `KBytes()`, … | convenience wrappers that call `dimensionCount` with the appropriate constant |
| `HumanReadableString(flags uint8)` | returns a string like “12.345 MiB” for a given flag mask |
| `PreciseString(flags uint8)` | similar but uses integer division when the value is an exact multiple of the dimension |

---

## Methods on public types

* **`(*BitRate) UnmarshalText(text []byte)`** – parses strings such as “12 Mbit/s” into a `BitRate`.  
  *TODO*: author questions whether this logic is optimal.
* **`(BitRate) MarshalText()`** – serialises the value back to a byte slice.
* **`(*ByteSize) UnmarshalText(text []byte)`** – analogous for bytes.
* **`(ByteSize) MarshalText()`, `HumanReadable()`, `HumanReadableDec()`, `HumanReadableBin()`** – formatting helpers.

---

## Constructors

```go
func NewByteSize(bytes uint64) ByteSize
func NewBitRate(bitsPerSec uint64) BitRate
```

Convenience wrappers that create the public types from a raw value.

---

## Test coverage (datasize_test.go)

| Test | What it checks |
|------|----------------|
| `TestBitRate_UnmarshalText` | round‑trip parsing, marshaling and human‑readable output for bit rates. |
| `TestByteSize_UnmarshalText` | same for byte sizes. |

The tests use hard‑coded vectors such as `"300 Mbit/s"`, `"3445 Mbit/s"` etc., ensuring that both decimal and binary prefixes are handled correctly.

---

## Environment variables / flags

* **Flag masks** – the constants `bitDimFlag`, `byteDimFlag`, `decimalDimFlag` and `binaryDimFlag` can be combined to indicate which dimension a string refers to.  
  Example: `byteDimFlag | decimalDimFlag` selects a decimal byte prefix (e.g. “MB”).  
* **Configuration file** – the package does not currently read external config files, but the constants could be overridden by environment variables if desired.

---

## Edge cases / launch scenarios

Although this is a library, it can be used in several ways:

1. **As a pure Go import** – other packages can call `NewByteSize`, `UnmarshalText` and formatting helpers.
2. **CLI helper** – a small wrapper program could read a file or stdin containing data‑size strings, parse them with `BitRate.UnmarshalText()`/`ByteSize.UnmarshalText()`, then output the human‑readable form.  
   *Command line arguments* that might be useful:  
   - `-i <input>` – path to a text file of sizes.  
   - `-o <output>` – destination for formatted results.  
3. **Configuration** – if the package is compiled into a larger tool, the constants (`fnUnmarshalText`, flag masks) could be overridden via build tags or environment variables.

---

## Summary

`util/datasize` provides:

* A numeric base type `bitSize`.  
* Two public wrappers (`ByteSize`, `BitRate`) that expose parsing, marshaling and human‑readable formatting.  
* Helper functions for dimension handling and flag masks.  
* Unit tests that confirm round‑trip correctness for typical data‑size strings.

The code is self‑contained; the only external dependency is the *testify* package used in tests.