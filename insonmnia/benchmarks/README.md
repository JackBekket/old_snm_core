# Benchmarks Package

## Short overview  
The `benchmarks` package provides a lightweight loader for benchmark definitions that are stored as JSON objects.  
* A **configuration** (`Config{URL string}`) points to an HTTP(S) endpoint or a local file.  
* The loader parses the URL, fetches the data, decodes it into three internal maps (by ID, by code, by device type), and exposes accessor methods for fast lookup.  
* A second module (`mapping.go`) turns that list into either an array‑based or map‑based *Mapping* structure depending on the size of the benchmark set.

---

## Environment / configuration

| Source | Key | Description |
|--------|-----|-------------|
| `Config{URL}` | `benchmarks.URL` | The JSON source location. Can be supplied via: |
| | env var | `BENCHMARKS_URL` (optional) |
| | flag | `-url <uri>` (optional) |

The package can also be configured by passing a command‑line argument or an environment variable; the code currently expects only the URL, but additional flags could be added later.

---

## Project file structure

```
insonmnia/
└── benchmarks/
    ├── benchmarks.go
    └── mapping.go
```

* `benchmarks.go` – core loader and data structures.  
* `mapping.go` – helper that turns a `BenchList` into an efficient lookup table.

---

## Key code entities & their relations

| File | Entity | Purpose |
|------|---------|---------|
| `benchmarks.go` | `Config{URL string}` | Holds the source location. |
| | `BenchList` interface | Declares `Max()`, `ByID()`, `MapByDeviceType()`, `MapByCode()` |
| | `benchmarkList` struct | Implements `BenchList`; holds three maps (`byID []sonm.Benchmark`, `byCode map[string]uint64`, `byType map[sonm.DeviceType][]uint64`). |
| | `load(ctx, s)` | Entry point that parses the URL and delegates to either `loadURL` or `loadFile`. |
| | `readResults(ctx, r io.ReadCloser)` | Decodes JSON into a temporary map, then populates all three internal maps. |
| | `NewBenchmarksList(ctx, cfg)` | Public constructor; returns a `BenchList`. |
| | Accessor methods (`Max`, `ByID`, etc.) | Provide read‑only views of the internal data. |

| File | Entity | Purpose |
|------|---------|---------|
| `mapping.go` | `Loader` interface | Declares `Load(ctx) (Mapping, error)` |
| | `loader` struct | Holds a URI; used by `NewLoader`. |
| | `NewLoader(uri string)` | Factory that returns a `Loader`. |
| | `(*loader).Load(ctx)` | Calls `NewBenchmarksList`, decides on array vs map mapping, and returns a concrete `Mapping`. |
| | `NewArrayMapping(benchmarks BenchList, maxID uint64)` | Builds two slices (`deviceTypes`, `splittingAlgorithms`) sized to the maximum ID. |
| | `NewMapMapping(benchmarks BenchList)` | Builds two maps keyed by benchmark ID. |
| | `Mapping` interface | Declares `DeviceType(id int) sonm.DeviceType` and `SplittingAlgorithm(id int) sonm.SplittingAlgorithm`. |
| | `mapping` struct | Map‑based implementation of the interface. |
| | `arrayMapping` struct | Slice‑based implementation; used when IDs are small enough for efficient array access. |

**How they interact**

1. A client creates a `loader` via `NewLoader(uri)` and calls `Load(ctx)`.  
2. Inside `Load`, `NewBenchmarksList` reads the JSON source into a `benchmarkList`.  
3. The loader then checks the maximum ID (`maxID`) and chooses either an array‑based or map‑based mapping.  
4. The chosen mapping type implements the same interface, so callers can use it interchangeably.

---

## Launch edge cases

* **HTTP(S) source** – If `Config.URL` starts with `http://` or `https://`, `loadURL` will perform an HTTP GET and return a response body for decoding.  
* **Local file source** – If the scheme is `file://`, `loadFile` opens the local path via `os.Open`.  
* **CLI usage** – A simple main program could look like:

```go
func main() {
    ctx := context.Background()
    cfg := benchmarks.Config{URL: os.Getenv("BENCHMARKS_URL")}
    loader := benchmarks.NewLoader(cfg.URL)
    mapping, err := loader.Load(ctx)
    if err != nil { log.Fatal(err) }
    // use mapping.DeviceType(42), etc.
}
```

The program can be compiled with `go build ./...` and run as a command‑line tool; the URL may also be passed via a flag or env variable.

---

All together, this package implements a complete pipeline: fetch → decode → store → map. It is ready to be used by other parts of the system that need benchmark data for device type and splitting algorithm lookups.