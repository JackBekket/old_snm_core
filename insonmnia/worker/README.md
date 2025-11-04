# Package `worker`

The **`worker`** package implements a full Docker‑based task executor that is driven by gRPC, SSH and a rich configuration hierarchy.  
At the top level it exposes a `Worker` type (in *server.go*) that owns an overseer, a set of containers, a storage backend and a handful of helper objects for networking, GPU handling, metrics and authorization.  The package is split into logical sub‑packages (`gpu`, `network`, `plugin`, `volume`, etc.) but all files live under the same module root.

---

## File structure

```
insonmnia/worker/
├─ acl.go
├─ acl_test.go
├─ cgroup_test.go
├─ config.go
├─ config_test.go
├─ container.go
├─ docker.go
├─ geoip.go
├─ options.go
├─ overseer.go
├─ overseer_test.go
├─ server.go
├─ server_test.go
├─ ssh.go
├─ ssh_test.go
├─ util.go
├─ whitelist.go
└─ whitelist_test.go
```

---

## What the package does

* **Configuration** – `config.go` loads a YAML file into a rich struct that contains endpoint URLs, logging settings, cgroup limits, GPU drivers, network specs, plugin lists, storage paths and more.  
  * The config is read by `NewConfig(path string)`; it can be overridden via an environment variable (e.g. `WORKER_CONFIG`) or a command‑line flag (`-config`).  

* **Authorization** – `acl.go` defines a small chain of authorizers that extract a deal ID from gRPC metadata, compare the wallet hex string and combine multiple checks with `anyOfAuth / allOfAuth`.  Tests in *acl_test.go* verify this logic.  

* **Container lifecycle** – `container.go` implements Docker container creation, start/stop, exec commands over SSH and extended network statistics collection.  The helper structs (`NetworkStatsExt`, `containerDescriptor`) tie together Docker API calls with the overseer’s plugin repository.  

* **GPU handling** – The sub‑folder `gpu/` contains drivers for NVIDIA, Radeon and a generic tuner; it is used by the container creation logic to expose GPU devices inside containers.  

* **Networking** – `network/*.go` implements L2TP networking, IPAM, tuners and a manager that can be started on Linux or non‑Linux hosts (`manager_linux.go`, `manager_nonlinux.go`).  The overseer uses this to attach containers to the correct network stack.  

* **SSH server** – `ssh.go` builds an SSH listener that accepts commands from the overseer, forwards them into a container and returns logs via gRPC.  It is started by `setupServer()` in *server.go*.  

* **Metrics & storage** – The package uses `metrics/metrics.go`, `storage/btrfs/*.go` and `volume/*.go` to persist container state, upload images and mount volumes.  

---

## Environment variables / flags / command‑line arguments

| Variable / flag | Purpose |
|------------------|---------|
| `WORKER_CONFIG` (env) | Path of the YAML config file; default is `"worker.yaml"`.  The package reads it via `NewConfig(os.Getenv("WORKER_CONFIG"))`. |
| `-config <path>` (flag) | Same as above, but can be passed to a CLI wrapper that starts the worker. |
| `-loglevel <lvl>` (flag) | Sets the zap logger level used by the overseer and SSH server. |
| `-metricsAddr <addr>` (flag) | Overrides the metrics listening address defined in config; useful for debugging. |

---

## Edge cases of launching

1. **Standalone worker** – Run a binary that calls `worker.NewWorker()` with the parsed config, then `setupServer()` to expose gRPC endpoints.  The worker can be started via a simple shell script or as a systemd service.  
2. **Docker image pre‑build** – The test harness in *overseer_test.go* shows how an image named `"worker"` is built from a tar archive; the same logic can be used in production to pull images from a registry before starting containers.  
3. **SSH command execution** – `ssh.go` exposes a handler that accepts SSH sessions over gRPC; this allows external tools (e.g. a CI pipeline) to run arbitrary commands inside a container and stream logs back to the worker.  

---

## Summary of key code entities

* **`Config`** – central struct loaded by `NewConfig`; contains nested structs for logging, network, GPU, plugin, volume, etc.  
* **`Worker`** – owns an overseer, storage, SSH client and a map of running containers; it exposes methods like `PushTask`, `StartTask`, `StopTask`.  
* **`Overseer`** – manages Docker containers: create, start, exec, upload, metrics collection.  It uses the plugin repository to tune container specs (`gpu/dri.go`, `network/l2tp_tuner.go`).  
* **`Description`** – a lightweight wrapper around a protobuf `Container`; it holds environment variables, GPU device lists and network options that are passed into Docker when creating a container.  
* **`acl.go`** – defines the authorization chain; the key functions are `newContextDealExtractor`, `newFromNamedTaskDealExtractor`, `newDealAuthorization`, `anyOfAuth`, `allOfAuth`.  These are exercised in *acl_test.go*.  

---

## How code entities relate

1. **Config → Worker** – `NewWorker` calls `init()` which loads config, creates an overseer and starts the SSH server.  
2. **Overseer → ContainerDescriptor** – `newContainer` builds a Docker container spec from a `Description`, tunes it via the plugin repo (`gpu/dri.go`) and stores the resulting ID in a `containerDescriptor`.  
3. **SSH ↔ Overseer** – The SSH listener created by `ssh.go` forwards commands to the overseer; the overseer then calls `Exec()` on a container, which uses Docker exec API and streams logs back via gRPC.  
4. **Metrics → Worker** – `metrics/metrics.go` collects CPU/memory/network stats from containers; these are exposed through `Worker.CollectTasksStatuses`.  

---

## Edge cases of launching

* **CLI main package** – A small wrapper binary can be built that parses a config file, creates a `worker.Worker`, starts the overseer and listens on gRPC.  The binary could accept flags like `-config` or `-loglevel`.  
* **Docker image pre‑build** – The test harness in *overseer_test.go* shows how to build an image named `"worker"` from a tar archive; this logic can be reused in production to pull images from a registry before starting containers.  
* **SSH command execution** – `ssh.go` exposes a handler that accepts SSH sessions over gRPC; this allows external tools (e.g. a CI pipeline) to run arbitrary commands inside a container and stream logs back to the worker.  

---

## Summary of what the package code does

The `worker` package is a full Docker‑based task executor that:

* Loads a YAML configuration file into a rich struct (`Config`).  
* Provides an authorization chain for deal IDs and KYC checks (`acl.go`).  
* Creates, starts, stops and uploads Docker containers via an overseer (`overseer.go`).  
* Exposes SSH commands over gRPC (`ssh.go`) and collects extended network statistics.  
* Uses a plugin repository to tune GPU devices, networking and volume mounts.  

The package can be launched as a standalone binary that reads its config from an env var or flag, starts the overseer, exposes an SSH server and listens for gRPC calls.