# salesman

## Short summary  
The **salesman** package implements a full‑stack orchestrator for ask‑plans, deals and orders on an Ethereum‑based blockchain.  
* `options.go` defines the functional option pattern that builds a configuration struct (`options`) used by the main `Salesman` type in `salesman.go`.  
* The main file contains the core logic: construction of the component, CRUD for ask‑plans, background sync with the chain, provisioning of cgroups and networks, deal lifecycle handling, order placement and maintenance scheduling.  

The package is ready to be used as a library or launched directly from a CLI command.

---

## Project structure

```
insonmnia/worker/salesman/
├── options.go
└── salesman.go
```

---

## Environment variables / flags / config files that can be supplied

| Source | Variable / flag / file | Purpose |
|--------|-----------------------|---------|
| **Env** | `SALESMAN_LOGGER` | zap logger instance (or default) |
| | `SALESMAN_STORAGE_PATH` | path to persistent state store |
| | `SALESMAN_RESOURCE_SCHEDULER` | resource scheduler config |
| | `SALESMAN_HARDWARE_CONFIG` | hardware abstraction settings |
| | `SALESMAN_BLOCKCHAIN_ENDPOINT` | Ethereum node endpoint |
| | `SALESMAN_CGROUPS_MANAGER` | cgroup manager config |
| | `SALESMAN_MATCHER_CONFIG` | matcher component config |
| | `SALESMAN_ECDSA_KEY_FILE` | path to ECDSA private key file |
| **Flags** | `--log-level` | zap log level |
| | `--storage-path` | same as env above |
| | `--yaml-config-file` | YAML configuration for durations, sync intervals etc. |
| | `--network-config-file` | network manager config file |
| **Config files** | `config.yaml` (YAMLConfig) – path: `insonmnia/worker/salesman/config.yaml` |
| | `network.json` – path: `insonmnia/worker/salesman/network.json` |

> *All of the above are optional; the functional options in `options.go` provide defaults and validation.*

---

## How the application can be launched

1. **As a library**  
   ```go
   sm := salesman.NewSalesman(
       salesman.WithLogger(zap.NewExample()),
       salesman.WithStorage(myStore),
       salesman.WithResourceScheduler(rSched),
       salesman.WithHardware(hard),
       salesman.WithBlockchainAPI(bcAPI),
       salesman.WithCGroupManager(cgm),
       salesman.WithMatcher(matcher),
       salesman.WithECDSAKey(key),
       salesman.WithYAMLConfig(yamlCfg),
       salesman.WithNetworkConfig(netCfg),
   )
   sm.Run(ctx)
   ```

2. **As a CLI command**  
   * Build the binary `go build -o bin/salesman ./insonmnia/worker/salesman`  
   * Run with optional flags:  
     ```bash
     ./bin/salesman \
       --log-level=debug \
       --storage-path=/var/lib/salesman \
       --yaml-config-file=config.yaml \
       --network-config-file=network.json
     ```

3. **Edge cases**  
   * If no functional options are supplied, `options.Validate()` will error out – the package guarantees all required fields are present before use.  
   * The component can be started with a pre‑existing network manager (via `WithNetworkManager`) or let it create one internally (`NewSalesman`).  
   * The sync routine can be disabled by passing `salesman.WithSyncInterval(0)` if the user wants to run only manually.

---

## Relations between code entities

| Entity | Relationship |
|--------|--------------|
| `options` (in options.go) | Holds all dependencies; embedded in `Salesman`. |
| `With…` functions | Functional option helpers that set fields of `options`; used by `NewSalesman`. |
| `Validate()` | Aggregates missing‑field errors from the functional options. |
| `Salesman.NewSalesman` | Creates a new instance, builds a network manager (`networkManager`) and loads existing ask‑plans into memory. |
| `CreateAskPlan`, `RemoveAskPlan` | CRUD helpers that manipulate the in‑memory maps (`askPlans`, `askPlanCGroups`, `askPlanNetworks`). |
| `syncRoutine` → `syncWithBlockchain` → `syncPlanWithBlockchain` | Background loop that keeps each plan in sync with the blockchain and triggers order placement / deal creation. |
| `createCGroup`, `dropCGroup`, `createNetwork`, `dropNetwork` | Dedicated helpers for provisioning cgroups/networks per plan. |
| `placeOrder`, `waitForDeal` | Build an order from a plan, place it on chain and wait until the matcher creates a deal. |
| `ScheduleMaintenance`, `NextMaintenance` | Compute next maintenance time for a plan; used by `placeOrder`. |

---

## Unclear places / dead code

*The current files contain no obvious dead code or missing references.*  
*All functional options are exercised in `NewSalesman`; the only TODO left is `restoreState()` which will be implemented later.*

---