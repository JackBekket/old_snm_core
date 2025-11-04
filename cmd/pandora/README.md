# pandora

The **pandora** package implements a lightweight orchestration layer for interacting with a data‑warehouse (DWH) and a marketplace service.  
It defines two distinct *ammo* types – one that pulls orders from the DWH, another that queries/places orders on the marketplace – together with factories that create these ammo objects from configuration.  
A **gun** component wraps an aggregator and executes any ammo instance while reporting results via a network sample.  
The package also provides a global registry for registering ammo‑factory constructors, a provider that pulls ammo from a channel and returns it to the gun, and a bootstrap `main.go` that wires everything together and starts the CLI.

---

## Environment variables, flags & command‑line arguments

| Variable / Flag | Purpose |
|-----------------|---------|
| *None explicitly defined* | The package relies on configuration files read by `importer.Import(fs)` in `cmd/pandora/main.go`.  These files are expected to contain keys such as `"sonm.marketplace.GetOrderInfo"`, `"sonm.DWH.Orders"` and provider details under the key `"detail"`. |
| CLI arguments | The binary is started via `go run ./cmd/pandora` (or built with `go build`).  The CLI itself is provided by the imported package `github.com/yandex/pandora/cli`; it consumes the registered guns, ammo factories and provider. |

---

## Project file structure

```
cmd/pandora/
├── ammo.go
├── ammo_dwh.go
├── ammo_marketplace.go
├── common.go
├── config.go
├── gun.go
├── gun_dwh.go
├── gun_marketplace.go
├── main.go
├── provider.go
└── registry.go
```

---

## Relations between code entities

| Entity | Role | Connected components |
|--------|------|---------------------|
| `AmmoType` (ammo.go) | Identifier for ammo objects | Used by all factories and the provider’s pool array |
| `Ammo`, `AmmoFactory`, `PoolAmmoFactory` | Core abstraction for creating & pooling ammo | Implemented in `ammo_dwh.go`/`ammo_marketplace.go`; used by `provider.go` |
| `DWHOrdersAmmo`, `OrderInfoAmmo`, `OrderPlaceAmmo` | Concrete ammo types | Created by their respective factories (`dwhOrdersAmmoFactory`, `orderInfoAmmoFactory`, `orderPlaceAmmoFactory`) |
| `gun` (gun.go) | Executes an ammo and reports a network sample | Bound to an aggregator via `Bind`; used by the CLI after registration in `main.go` |
| `dwhExt`, `marketplaceExt` | Runtime state for DWH and marketplace guns | Created in `gun_dwh.go`/`gun_marketplace.go` and passed into `newGun` |
| `provider` (provider.go) | Pulls ammo from a channel, pools it, and hands it to the gun | Uses the global registry (`AmmoRegistry`) to obtain factories; its `Run` method feeds ammo into the channel |
| `AmmoRegistry` (registry.go) | Global map of factory constructors keyed by string | Populated in `main.go` with calls such as `AmmoRegistry.Register("sonm.DWH.Orders", newDWHOrdersAmmoFactory)` |

---

## Edge cases for launching

* **CLI entry point** – The binary is started via the `cli.Run()` call in `cmd/pandora/main.go`.  It expects that all ammo factories and guns have been registered beforehand.
* **Configuration loading** – The importer reads configuration files from the OS filesystem; if any required key (e.g. `"detail"`) is missing, the provider will return an error during construction.
* **Provider limits** – `Config.AmmoLimit` controls how many ammo objects are produced; if set too low, some guns may not receive enough work.

---

## Summary of package logic

1. **Factories** create ammo objects for DWH orders and marketplace operations, each backed by a `sync.Pool`.  
2. **Provider** pulls these ammo objects from its channel, pools them by type, and hands them to the gun.  
3. **Gun** executes any ammo instance, logs success/failure, and reports a network sample via an aggregator.  
4. **Main bootstrap** registers all factories and guns in global registries, builds a provider with configuration, and starts the CLI.

This structure allows adding new ammo types or guns simply by registering another factory and gun in `main.go`.