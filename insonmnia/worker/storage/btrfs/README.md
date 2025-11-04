# btrfs

## Overview  
The `btrfs` package implements a thin Go wrapper around the Linux **BTRFS** quota‑group API.  It exposes two concrete implementations of the same interface:

| Implementation | Purpose |
|-----------------|---------|
| `btrfsCLI` – in *btrfs_api.go* | Executes the external `btrfs` binary via `exec.CommandContext`.  All high‑level quota operations (enable, create, destroy, limit, assign, remove) are implemented as simple shell calls. |
| `btrfsNativeAPI` – in *btrfs_native_api.go* | Uses cgo to call the kernel’s ioctl interface directly (`BTRFS_IOC_*`).  It performs the same operations but without spawning a process, making it faster and more deterministic for unit tests. |

Both implementations satisfy the same `API` interface defined in *btrfs_api.go*.  The test files exercise each implementation separately and together with an end‑to‑end helper (`testE2EOne`) that verifies a full quota lifecycle.

---

## Environment variables & configuration  

| Variable | Meaning |
|----------|---------|
| `BTRFS_PLAYGROUND_PATH` | Base directory used by the integration tests (e.g. `/mnt/btrfs`).  The value is read in *btrfs_native_api_test.go* and *cliapi_test.go*. |
| `SUDO_USER` | Optional flag that indicates root privileges are available for the E2E test; if unset the test is skipped. |

No command‑line flags or build tags are required beyond the standard Go build system.

---

## Files & project structure  

```
insonmnia/worker/storage/btrfs/
├── btrfs_api.go                # API interface + CLI implementation
├── btrfs_api_test.go           # Tests for the CLI implementation
├── btrfs_native_api.go         # Native (cgo) implementation
├── btrfs_native_api_test.go    # Tests for the native implementation
└── cliapi_test.go              # Helper tests + E2E test that drives both implementations
```

---

## Key code entities & their relationships  

| Entity | File | Role |
|--------|-------|------|
| `API` interface | *btrfs_api.go* | Declares all quota‑group operations. |
| `NewAPI()` | *btrfs_api.go* | Factory that returns a `btrfsCLI`. |
| `btrfsCLI` struct | *btrfs_api.go* | Implements the `API` interface via methods such as `QuotaEnable`, `QuotaCreate`, etc. |
| `btrfsNativeAPI` struct | *btrfs_native_api.go* | Implements the same interface but uses ioctl calls. |
| `quotaCreateOrDestroy()` | *btrfs_native_api.go* | Shared helper for create/destroy operations. |
| `lookupQuotaInShowOutput()` / `lookupIDForSubvolumeWithPath()` | *cliapi_test.go* | Parsing helpers used by the tests to validate command output. |
| `testE2EOne()` | *btrfs_native_api_test.go* | End‑to‑end helper that exercises a full quota lifecycle (enable → create → assign → limit → write). |

The test files use the same interface, so they can be run against either implementation by swapping the concrete type (`btrfsCLI` vs `btrfsNativeAPI`).  The CLI tests focus on shell‑based commands; the native tests focus on ioctl calls.  Both share a common helper (`testE2EOne`) that performs the same sequence of operations, ensuring both implementations are functionally equivalent.

---

## How to launch / use the package  

1. **As a library** – Import `github.com/insomnnia/worker/storage/btrfs` and call `NewAPI()` or instantiate `btrfsNativeAPI{}` directly.  
2. **Command‑line usage** – The CLI implementation can be used by any higher‑level tool that needs to manipulate BTRFS quota groups; e.g.:

   ```bash
   # Enable a quota group on /mnt/btrfs
   btrfs quota enable /mnt/btrfs

   # Create a new qgroup
   btrfs qgroup create 1/999 /mnt/btrfs
   ```

3. **Testing** – Run `go test ./...` in the repository root.  The tests automatically pick up the environment variables above and will execute both implementations.

---

## Edge cases & assumptions  

* The code assumes that the external binary `btrfs` is available on `$PATH`.  
* All ioctl calls use the same file descriptor (`getDirFd`) as a base; if this fails, the native implementation will error out.  
* The tests expect at least one sub‑volume to exist after `QuotaCreate`; if it does not, subsequent operations (e.g. `GetQuotaID`) will fail.  

---

## Summary of logic flow  

1. **API definition** – *btrfs_api.go* declares the contract.  
2. **CLI implementation** – each method builds a command string, runs it via `exec.CommandContext`, logs output with `zap` and returns any error.  Helper functions (`lookupQuotaInShowOutput`, etc.) parse stdout into Go values.  
3. **Native implementation** – uses cgo to call the kernel directly; helper structs (`btrfsCreateFlag`, etc.) wrap ioctl flags.  All methods follow a similar pattern: build a C struct, perform `unix.Syscall`, check return status, and log errors.  
4. **Tests** – verify parsing helpers and run an end‑to‑end sequence that covers all operations for both implementations.

---