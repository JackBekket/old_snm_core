# storage

The **storage** package implements a small abstraction around Docker‑based BTRFS quota tuning and exposes a test harness that demonstrates its usage.  
It is split into three logical layers:

| Layer | File(s) | Purpose |
|-------|---------|---------|
| **API wrapper** | `btrfs/` | Thin wrappers over the low‑level BTRFS API (`btrfs_api.go`, `btrfs_native_api.go`) and a test harness (`btrfs_quota_test.go`). |
| **Quota tuner** | `btrfs_quota_tuner.go` | Implements the public interface `StorageQuotaTuner`. It creates a quota group, assigns it to a container, and returns a cleanup object. |
| **Factory & helpers** | `common.go`, `errors.go`, `quotafactory_linux.go`, `quotafactory_nonlinux.go` | Declares the core types (`btrfsQuotaTuner`, `btrfsQuotaCleaner`) and the factory function that chooses the correct tuner implementation based on the Docker driver. |

---

## Environment variables, flags & command‑line arguments

| Variable / Flag | Description |
|------------------|-------------|
| `SUDO_USER` | Used by the test to decide whether it should run (requires sudo privileges). |
| `docker client` | The test creates a Docker client via `client.NewEnvClient()`. |
| `busybox` image | Base image for all containers created in the test. |
| Command string | `"dd if=/dev/zero of=/FILE bs=1024 count=10000"` – generates a 10 000‑block file inside each container. |

---

## How the package can be launched

* **As a library** – Import `github.com/sonm-io/core/insonmnia/worker/storage` in another Go module and call `NewQuotaTuner(info)` to obtain a tuner that can set quotas on containers.
* **As a test harness** – Run `go test ./...` from the repository root. The file `btrfs_quota_test.go` contains a single test that creates three containers, applies a quota of 20 MiB each and verifies that the resulting files are correctly copied back to the host.

---

## File structure

```
insonmnia/worker/storage/
├─ btrfs/
│   ├─ btrfs_api.go
│   ├─ btrfs_api_test.go
│   ├─ btrfs_native_api.go
│   ├─ btrfs_native_api_test.go
│   └─ cliapi_test.go
├─ btrfs_quota_test.go
├─ btrfs_quota_tuner.go
├─ common.go
├─ errors.go
├─ quotafactory_linux.go
└─ quotafactory_nonlinux.go
```

---

## Core logic & relations

### 1. `btrfs_quota_tuner.go`
* **`btrfsQuotaTuner`** – holds Docker root dir, sub‑volume path and embeds the BTRFS API.
* **`SetQuota(ctx, ID, quotaID, bytes)`** –  
  * Reads the mount ID for a container layer.  
  * Creates/ensures a quota group (`qgroupID`) using an FNV hash of `quotaID`.  
  * Calls `btrfs.API.QuotaExists`, `QuotaLimit` and `GetQuotaID` to bind the group to the container.  
  * Returns a `btrfsQuotaCleaner` that can later remove the quota.

### 2. `common.go`
* Declares the public interface `StorageQuotaTuner` with method `SetQuota`.  
* Provides a small struct `QuotaDescription` and an interface `Cleanup` for cleanup objects.

### 3. `errors.go`
* Defines `ErrDriverNotSupported`, used by the factory to signal unsupported drivers.

### 4. `quotafactory_linux.go` / `quotafactory_nonlinux.go`
* Build tags ensure that only one of these files is compiled per OS.  
* The Linux version implements `NewQuotaTuner(info)` which dispatches to `newBtrfsQuotaTuner`.  
* The non‑Linux stub currently returns an error; it can be extended for other drivers.

### 5. Test harness (`btrfs_quota_test.go`)
* Creates a Docker client, obtains the tuner via `NewQuotaTuner`, and creates three containers named “aaa”, “bbb” and “ccc”.  
* For each container it calls `tuner.SetQuota` with a limit of 20 MiB.  
* After starting all containers it copies back `/FILE` from each one, verifies that exactly one copy succeeded and that the total size is within expected bounds.

---

## Edge cases & potential extensions

| Edge case | Explanation |
|-----------|-------------|
| **Missing `SUDO_USER`** | The test skips if not set; ensure sudo privileges when running tests. |
| **Non‑Linux platforms** | Currently only Linux is supported; the non‑Linux file should be implemented to provide a fallback tuner. |
| **Quota group collision** | `SetQuota` checks for existence before creating a new group, preventing duplicate groups on repeated runs. |

---

**<end_of_output>**