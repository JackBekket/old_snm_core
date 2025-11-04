# Plugin Package – `plugin`

The **`plugin`** package implements a lightweight Docker‑based worker that can create, tune and clean up volumes, GPUs and overlay networks.  
It is organized around three source files:

```
insonmnia/worker/plugin/
├── cleanup.go
├── config.go
└── plugin.go
```

---

## 1. What the package does

| File | Key responsibilities |
|------|-----------------------|
| **cleanup.go** | Defines a `Cleanup` interface and two concrete implementations (`nestedCleanup`, `volumeCleanup`). The former aggregates child clean‑ups; the latter removes a Docker volume via a `volume.VolumeDriver`. |
| **config.go** | Declares the configuration data model that is read from YAML. It contains: <br>`SocketDir` – where plugin sockets live, <br>`Volumes` – root directory and driver map for Docker volumes, <br>`Overlay` – Tinc & L2TP overlay drivers, <br>`GPUs` – a two‑level GPU configuration map. |
| **plugin.go** | Implements the core logic: building a `Repository`, orchestrating tuning of GPUs, volumes and networks, and providing cleanup objects for each step. It also exposes helper methods (`TuneGPU`, `TuneVolumes`, `TuneNetworks`) that are used by higher‑level workers. |

The package is intended to be invoked from a CLI or as part of a larger worker system; the main entry point would typically call `plugin.Tune(...)` with a configuration file and Docker client.

---

## 2. Environment variables, flags & command‑line arguments

| Source | Variable / flag | Default / usage |
|--------|-----------------|-----------------|
| **Config** | `SocketDir` | `/run/docker/plugins` (overridden by YAML key `socket_dir`) |
| | `Volumes.Root` | `/var/lib/docker-volumes` (YAML key `volume`) |
| | `Overlay.Tinc` / `Overlay.L2TP` | keys `tinc`, `l2tp` in YAML |
| | `GPUs` | map of GPU vendor → driver options (keyed by `gpu`) |

Typical CLI usage:

```bash
# Build the plugin binary
go build -o bin/plugin ./insonmnia/worker/plugin

# Run it with a config file
./bin/plugin --config=plugin.yaml
```

The binary would read `plugin.yaml`, populate the `Config` struct, create a `Repository`, and call `Tune(ctx, provider, hostCfg, netCfg)`.

---

## 3. File structure (project package)

```text
insonmnia/worker/plugin/
├── cleanup.go          # interface & nested cleanup logic
├── config.go           # Config structs + YAML tags
└── plugin.go           # Repository, tuning orchestration, helpers
```

All files belong to the same Go module `plugin`. The package imports:

* Docker client (`github.com/docker/docker/client`)
* Docker types (`container`, `network`)
* Logging (`go.uber.org/zap` via alias `zapctx/ctxlog`)
* Core modules: `hardware`, `structs`, `gpu`, `storage`, `volume`
* Network overlay helpers from `worker/network`

---

## 4. How the code entities relate

1. **Repository** holds maps of drivers (`volumes`, `gpuTuners`, `networkTuners`) and a storage‑quota tuner.
2. `NewRepository(cfg)` builds this map by iterating over the configuration: it creates volume drivers, GPU tuners (for each vendor), overlay network tuners (Tinc & L2TP) and optionally a quota tuner if Docker supports it.
3. `Tune(ctx, provider, hostCfg, netCfg)` orchestrates the tuning steps:
   * `TuneGPU` – applies GPU settings to the host config via all GPU tuners.
   * `TuneVolumes` – creates volumes for each provider entry, mounts them and registers a `volumeCleanup`.
   * `TuneNetworks` – tunes overlay networks (Tinc & L2TP) using the network tuners.
4. Each tuning step returns a `Cleanup` object; they are chained via `nestedCleanup`. The final cleanup chain can be executed later to roll back or clean up all resources.

---

## 5. Edge cases for launching

| Scenario | What to consider |
|----------|------------------|
| **CLI main** – the package could expose a `main.go` that parses flags, loads YAML into `Config`, creates a Docker client and calls `plugin.Tune(...)`. |
| **Unit tests** – `EmptyRepository()` can be used in tests to create an empty repository before populating it with mock drivers. |
| **Error handling** – each `Close()` method aggregates errors; callers should check the returned error slice for failures. |

---

## 6. Summary

The `plugin` package is a small but complete worker that:

* Reads configuration from YAML (socket dir, volumes, overlay drivers, GPUs).  
* Builds a repository of Docker volume drivers, GPU tuners and overlay network tuners.  
* Orchestrates tuning of GPUs, volumes and networks in one pass, while collecting cleanup objects for rollback.  

It is ready to be invoked as part of a larger worker system or directly from the command line.