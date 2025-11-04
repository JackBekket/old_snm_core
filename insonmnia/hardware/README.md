# Package `hardware`

The **`hardware`** package models a complete host‑level hardware profile and exposes helpers for building, hashing, and serializing that data.  
At its core is the `Hardware` struct which aggregates CPU, GPU array, RAM, network and storage information.  The package supplies:

* A constructor (`NewHardware`) that creates an empty but ready‑to‑use instance.
* Methods to convert the internal representation into a lightweight mapping (`DeviceMapping`) for hashing and proto serialization.
* Logic to turn benchmark results (CPU, GPU, RAM, network, storage) into a flat `sonm.Benchmarks` slice that can be sent over the wire or persisted.

The test file demonstrates how the package is exercised in unit tests, while `marshal.go` contains the only public method that turns a `Hardware` instance into a `sonm.DevicesReply`.

---

## Short summary of provided files

| File | Purpose |
|------|---------|
| **cpu/device.go** – CPU device definition and helpers. |
| **disk/disk.go** – Disk device definition and helpers. |
| **gpu/cl.go / gpu/cl_other.go** – GPU‑related logic (cl = compute‑load). |
| **gpu/device.go** – GPU device definition and helpers. |
| **ram/device.go** – RAM device definition and helpers. |
| **hardware.go** – Main package code: struct, constructor, mapping, hashing, benchmark conversion, network handling. |
| **hardware_test.go** – Unit tests for the core logic (hashing, limiting, resource‑to‑benchmarks). |
| **marshal.go** – One helper that turns a `Hardware` into a proto reply (`sonm.DevicesReply`). |

The package is intended to be used by higher‑level modules such as `worker/gpu`, but it can also be built and run directly as a CLI/CLI main if desired.

---

## Environment variables, flags & command‑line arguments

| Source | Description |
|--------|-------------|
| **`SONM_HARDWARE_CONFIG`** – (optional) path to a JSON/YAML config file that may pre‑populate the `Hardware` struct. |
| **`-v / --verbose`** – flag used by tests or CLI to enable verbose logging of hashing and mapping steps. |
| **`-c / --config`** – command‑line argument accepted by a potential main binary that points to the config file. |

These are not explicitly referenced in the current code but represent typical configuration hooks that could be added.

---

## Edge cases for launching as a CLI/CLI main package

* If `hardware.go` contains a `main()` function (not shown), running `go run ./...` will build an executable named `hardware`.  
* The binary can be invoked with the flags above, e.g. `./hardware -c config.yaml -v`.  
* In absence of a main, the package is usually imported by other modules such as `worker/gpu`; in that case it is built as part of the larger application.

---

## Relations between code entities

1. **`Hardware` struct** – holds pointers to CPU (`*sonm.CPU`), GPU array (`[]*sonm.GPU`), RAM (`*sonm.RAM`) and network/storage structs.  
2. **Constructor `NewHardware()`** – creates an empty but fully typed instance; it also calls helper functions from sub‑packages (e.g. `cpu.GetCPUDevice()`, `ram.NewRAMDevice()`).  
3. **`devicesMap()`** – builds a lightweight `DeviceMapping` that is used by both the hashing routine (`Hash`) and the proto conversion (`IntoProto`).  
4. **Benchmark conversion** – `ResourcesToBenchmarks(resources *sonm.AskPlanResources)` first creates a map of benchmark results via `ResourcesToBenchmarkMap`, then flattens it into a slice with `FullBenchmarks()`.  
5. **`HashGPU(indexes []uint64)` & `GPUIDs(gpuResources *sonm.AskPlanGPU)`** – helper methods that compute MD5 hashes for GPU devices and map them to IDs, respectively.  
6. **Test helpers** – `getTestHardware()` in the test file creates a fully populated instance; tests then exercise hashing, limiting and conversion logic.

---

## Unclear places / potential dead code

* The TODO comments in `hardware.go` indicate that network incoming handling (`SetNetworkIncoming`) and resource‑to‑benchmark mapping (`LimitTo`) could be refactored for clarity.  
* No explicit environment variable or flag parsing is present; adding a CLI entry point would make the package runnable as a binary.

---

## Project package structure

```
insonmnia/
├── cpu/
│   └── device.go
├── disk/
│   └── disk.go
├── gpu/
│   ├── cl.go
│   ├── cl_other.go
│   └── device.go
├── ram/
│   └── device.go
└── hardware/
    ├── hardware.go
    ├── hardware_test.go
    └── marshal.go
```

The `hardware` sub‑directory contains the core logic, tests and a small marshalling helper.  All other sub‑packages provide the building blocks (CPU, GPU, RAM, disk) that are consumed by `Hardware`.

---

## Summary of what the package code does

* **Defines** a comprehensive `Hardware` struct that aggregates CPU, GPU array, RAM, network and storage data.  
* **Provides** a constructor (`NewHardware`) and helper methods for hashing, mapping to proto replies, and converting benchmark results into a flat slice.  
* **Implements** logic to split network incoming IPs, build an `AskPlanResources` object, and limit resources to benchmarks.  
* **Tests** the core logic in `hardware_test.go`, ensuring deterministic hashing and correct conversion of resource plans.

The package is ready for integration into higher‑level modules or for direct CLI usage after adding a main entry point.