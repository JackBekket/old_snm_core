# Package `network`

The **`network`** package implements a Docker‑plugin‑style networking stack that can be used by the *Insonmnia* worker.  
It contains two independent drivers – an L2TP driver and a Tinc driver – each of which exposes its own IPAM interface, state persistence, and a tuner that starts Unix socket listeners for the plugin sockets.

---

## 1. High‑level architecture

| Layer | Responsibility |
|-------|----------------|
| **State** (`l2tp_state.go`, `tinc_state.go`) | Holds all runtime data in BoltDB, provides lookup helpers, and serialises to JSON. |
| **Drivers** (`l2tp_ipam.go`, `l2tp_network.go`, `tinc_driver.go`, `tinc_ipam.go`, `tinc_network.go`) | Implement the Docker‑plugin API for network creation, endpoint handling, IPAM requests, and container orchestration. |
| **Tuners** (`l2tp_tuner.go`, `tinc_tuner.go`) | Create a Docker client, initialise both drivers, open Unix sockets, and expose a `Tuner` interface that can be called by the worker’s main entry point. |
| **Manager** (`manager.go`, `manager_linux.go`, `manager_nonlinux.go`, `manager_remote.go`) | Provide a high‑level abstraction for creating Docker networks, aliasing interfaces, shaping traffic (TC), and remote QoS handling. |

The package is therefore a *complete* networking stack that can be used as a Docker network driver or as a standalone worker.

---

## 2. Configuration sources

| File | Key | Description |
|------|-----|-------------|
| `config.go` | `remote_qos` | Path to a remote QoS server (used by the L2TP driver). |
| `l2tp_config.go` | `config` | Path to an L2TP network config file. |
| `tinc_config.go` | `enabled`, `config_dir`, `docker_net_plugin_dir`, `docker_ipam_plugin_dir`, `docker_image`, `state_path` | All Tinc‑specific settings (default values are hard‑coded). |

All structs use YAML tags, so the worker can load a single YAML file that contains all of these keys.

---

## 3. Environment variables / flags

* **Docker client** – created in each tuner (`client.NewEnvClient()`).
* **Unix socket paths** – `t.cfg.NetSocketPath` and `t.cfg.IPAMSocketPath` for the L2TP driver; `tinc_cfg.DockerNetPluginSockPath` and `tinc_cfg.DockerIPAMPluginSockPath` for the Tinc driver.
* **BoltDB bucket** – `"sonm_l2tp_driver_state"` (L2TP) or `"sonm_tinc_driver_state"` (Tinc).

---

## 4. File structure

```
insonmnia/worker/network/
├─ config.go
├─ l2tp_config.go
├─ l2tp_ipam.go
├─ l2tp_network.go
├─ l2tp_state.go
├─ l2tp_tuner.go
├─ manager.go
├─ manager_linux.go
├─ manager_nonlinux.go
├─ manager_remote.go
├─ tinc_config.go
├─ tinc_driver.go
├─ tinc_ipam.go
├─ tinc_network.go
├─ tinc_state.go
└─ tinc_tuner.go
```

---

## 5. Code relationships

* `l2tpState` (in *state.go*) holds a map of `Networks`.  
  Each network is an instance of `l2tpNetwork`, which in turn owns an endpoint (`l2tpEndpoint`).  
  The driver (`L2TPNetworkDriver`) embeds the state and exposes methods that operate on those structures.

* `parseOptsIPAM` / `parseOptsNetwork` (in *config.go*) read a `"config"` key from an IPAM or network request, unmarshal it into an `l2tpNetworkConfig`, validate it, and return the struct.  
  The returned config is used by `newL2tpNetwork` to initialise a new network.

* `IPAMDriver.RequestPool` creates a new L2TP network: it locks the state, parses options, builds a new network (`newL2tpNetwork`), registers it in the state and returns the pool ID.  
  The corresponding IPAM request is handled by Docker’s *go‑plugins‑helpers*.

* `TincNetworkDriver.CreateNetwork` does the same for Tinc: it creates a Docker container, starts tinc inside it, and stores the network in its own state (`TincNetworkState`).  
  The tuner (`NewTincTuner`) opens Unix sockets at the paths defined in *tinc_config.go* and serves both drivers.

* `manager_linux.go` implements concrete actions that are used by the local manager: aliasing a link, shaping traffic with TBF/HTB qdiscs, and creating IFB links.  
  The remote manager (`remoteNetworkManager`) in *manager_remote.go* performs the same operations via a QoS server.

---

## 6. Edge cases of launching

1. **L2TP driver** – Docker network creation is invoked with driver names `"l2tp_net"` and `"l2tp_ipam"`.  
   The tuner starts listeners on `t.cfg.NetSocketPath` and `t.cfg.IPAMSocketPath`, so the worker can call `NewL2TPTuner(ctx)` to start the plugin.

2. **Tinc driver** – Docker network creation is invoked with driver names `"tincipam"` and `"tincipam"`.  
   The tuner starts listeners on the sockets defined in *tinc_config.go* (`/run/docker/plugins/tinc/tinc.sock` etc.).  

3. **Remote QoS** – If a remote QoS server is configured, `manager_remote.go` will be used; otherwise the local manager from *manager_linux.go* handles all traffic‑control actions.

---

## 7. Summary of logic

1. **Configuration** – YAML files are parsed into structs (`NetworkConfig`, `l2tpNetworkConfig`, `TincNetworkConfig`).  
2. **State persistence** – BoltDB stores the entire state under a single key; helper methods `load()` and `sync()` keep it in sync with disk.  
3. **Drivers** – Each driver implements the Docker‑plugin API (`CreateNetwork`, `DeleteNetwork`, etc.) and uses its own IPAM interface to allocate pools and addresses.  
4. **Tuners** – The tuner creates a Docker client, opens Unix sockets for each plugin, starts goroutines that serve them, and exposes a `Tuner` interface that can be called by the worker’s main entry point.  
5. **Manager actions** – Concrete actions (aliasing, shaping, IFB creation) are defined in *manager_linux.go*; they are returned by `localNetworkManager.NewActions()` and executed when a network is created.

---

## 8. Edge cases & missing code

* The remote QoS server (`nilQOS`) in *manager_nonlinux.go* currently returns `ErrUnsupportedPlatform`; it should be implemented for non‑Linux platforms.  
* Several methods (e.g., `AllocateNetwork`, `DeleteEndpoint` in the L2TP driver) are stubs that need to be filled out later.

---

**<end_of_output>**