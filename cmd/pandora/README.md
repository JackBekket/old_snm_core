# pandora

The `cmd/pandora` directory implements a small, extensible “ammo‑factory” system that powers a command‑line tool.  
All files declare the same package (`main`) so they form one Go module that can be built and run as a single binary.

## Project structure
```
cmd/pandora/
├─ ammo.go
├─ ammo_dwh.go
├─ ammo_marketplace.go
├─ common.go
├─ config.go
├─ gun.go
├─ gun_dwh.go
├─ gun_marketplace.go
├─ main.go
├─ provider.go
└─ registry.go
```

## Environment variables, flags and command‑line arguments

| Variable / Flag | Purpose |
|------------------|---------|
| `LOG_LEVEL` (via `LoggingConfig.Level`) | Sets the verbosity of the Zap logger. |
| `ETH_ENDPOINT` (`EthereumConfig.Endpoint`) | RPC endpoint for Ethereum nodes. |
| `REGISTRY_ADDR` (`EthereumConfig.Registry`) | Address of the marketplace contract. |
| `DWH_ENDPOINT` (`DWHExtConfig.DWHEndpoint`) | gRPC endpoint for the data‑warehouse client. |

The binary is invoked as a normal Go program; no explicit command‑line flags are parsed in this module, but the `cli.Run()` call in `main.go` starts the whole workflow.

## How the application can be launched

1. **Build** – `go build ./cmd/pandora`.  
2. **Run** – execute the resulting binary (`./pandora`).  
   The program will:
   * Load configuration via `importer.Import(fs)`.
   * Register ammo factories for marketplace order‑info, order‑placement and DWH orders.
   * Register two guns (`sonm.marketplace`, `sonm.DWH`) that consume those factories.
   * Register a provider named `sonm` that supplies the gun with pre‑allocated ammo objects.
   * Finally call `cli.Run()` which triggers all registered components.

## Summary of package logic

### 1. Core types and interfaces (`ammo.go`)
* Defines an `AmmoType` identifier, an `Ammo` interface (extends `core.Ammo`) and a factory interface `AmmoFactory`.  
* Implements a simple pool‑based factory (`PoolAmmoFactory`) that wraps a `sync.Pool`.

### 2. DWH order ammo (`ammo_dwh.go`)
* Provides the concrete `DWHOrdersAmmo` struct, its `Execute` method (fetches orders from a data‑warehouse) and a factory `dwhOrdersAmmoFactory`.  
* The factory pulls requests from a configuration slice and reuses pooled objects.

### 3. Marketplace ammo (`ammo_marketplace.go`)
* Declares three ammo types: order info retrieval, order placement and DWH orders.  
* Implements `OrderInfoAmmo` (gets order data) and `OrderPlaceAmmo` (places an order).  
* Provides factories for each type that use a shared pool.

### 4. Common utilities (`common.go`)
* Holds global helpers for loading an ECDSA key, creating gRPC transport credentials and building a Zap logger.  
* These are used by the gun constructors in `gun_dwh.go` and `gun_marketplace.go`.

### 5. Gun abstraction (`gun.go`, `gun_dwh.go`, `gun_marketplace.go`)
* Defines a local `Gun` interface that extends `core.Gun`.  
* Implements a generic `gun` struct that stores an aggregator, external data and a logger.  
* The gun’s `Shoot` method executes any `Ammo` instance and reports the result to the aggregator.  
* Two concrete constructors (`NewDWHGun`, `NewMarketplaceGun`) create guns wired with DWH or marketplace extensions.

### 6. Provider abstraction (`provider.go`)
* Implements a pool‑based provider that keeps an array of `sync.Pool`s indexed by ammo type and a channel for hand‑off.  
* The provider is configured via a `Config` struct (limit, select string, detail slice).  
* It registers each factory in the global `AmmoRegistry`, builds pools, and returns a ready `core.Provider`.

### 7. Registry (`registry.go`)
* Provides a thread‑safe registry map that stores constructor functions for ammo factories.  
* The `Register` method adds a new entry; `Get` retrieves it.

## Relations between code entities

| Entity | Depends on | Notes |
|--------|------------|-------|
| `PoolAmmoFactory` | `sync.Pool` | Used by all concrete factory structs (`dwhOrdersAmmoFactory`, `orderInfoAmmoFactory`, etc.). |
| `ammo_dwh.go` / `ammo_marketplace.go` | `core.Ammo` | Provide concrete ammo types that implement `Execute`. |
| `gun_dwh.go` / `gun_marketplace.go` | `newGun(ext, log)` | Create guns that consume the above ammo types. |
| `provider.go` | `AmmoRegistry.Get` | Pulls factory constructors from the registry map. |
| `main.go` | `register.Gun`, `register.Provider` | Wire everything together and start execution. |

The code is intentionally modular: each file focuses on a single concern (ammo, gun, provider, registry). The global registry allows new ammo types to be added without touching other files.

---