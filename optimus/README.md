# optimus

## Project package structure  

```
optimus/
├── blacklist.go
├── cgroup.go
├── cgroup_linux.go
├── cgroup_nonlinux.go
├── config.go
├── context.go
├── devices.go
├── devices_test.go
├── engine.go
├── engine_axe.go
├── engine_branch.go
├── engine_genetic.go
├── engine_greedy.go
├── engine_multi.go
├── engine_test.go
├── knapsack.go
├── learning.go
├── learning_test.go
├── market.go
├── market_cache.go
├── matrix.go
├── model.go
├── model_lls.go
├── model_nnls.go
├── normalize.go
├── normalize_test.go
├── optimus.go
├── options.go
├── plan_policy.go
├── predictor.go
├── predictor_service.go
├── price.go
├── price_test.go
├── registry.go
├── tagging.go
├── watcher.go
└── worker.go
```

## Short summary of what the package does  

`optimus` is a modular optimisation framework that pulls together configuration, device‑management, market data, and several optimisation strategies (branch‑bound, genetic, greedy, axe, multi).  
At runtime it builds a *cgroup* for resource limits, loads a YAML config, fetches orders from a DWH service, consumes them into a knapsack, trains a regression model, and runs one of the optimisation engines to produce ask‑plans.  The package also contains a small cache layer for market data, a watcher that repeatedly triggers optimisation, and a registry that manages gRPC connections.

## Environment variables / flags / cmdline arguments  

| Variable / flag | Purpose |
|-----------------|----------|
| `OPTIMUS_CONFIG` | Path to the YAML file passed to `LoadConfig()` in *config.go* (default: `optimus.yaml`). |
| `-v, --verbose` | Enables verbose logging via the Zap logger. |
| `--interval` | Sets the market‑cache refresh interval (`marketCache.updateInterval`). |

The binary can be started with:

```bash
go run ./cmd/optimus -config optimus.yaml -v
```

or built into a binary named `optimus`.

## Edge cases of how application can be launched  

1. **Linux** – uses the implementation in *cgroup_linux.go* to create an LXC cgroup for the current process.  
2. **Non‑Linux** – falls back to *cgroup_nonlinux.go*, which currently returns a dummy deleter; this is useful on Windows or other OSes.  
3. **CLI main package** – a `main.go` in the root could call `optimus.NewOptimus()` and then `Run(ctx)` to start the optimisation loop.

## Relations between code entities  

* `blacklist` → used by *engine.go* for filtering orders that are already covered.  
* `DeviceManager` (in *devices.go*) is the core of the knapsack; it is created in *knapsack.go* and passed into every optimisation method.  
* The various engine files (`engine_axe.go`, `engine_branch.go`, etc.) all implement the same interface `OptimizationMethod`; they are instantiated by a factory defined in *model.go*.  
* `Model` (in *model_lls.go* / *model_nnls.go*) trains a regression model that is used by the optimizer.  
* The watcher (`watcher.go`) repeatedly triggers the engine via a ticker; it can be wrapped in a reactive or managed variant.  
* `registry.go` holds gRPC connections to the DWH and market services; it is used by *predictor_service.go* to fetch orders and submit ask‑plans.

## Unclear places / dead code  

No obvious dead code was detected after reviewing all files.  All functions are referenced either directly or via tests, so the package appears complete.

---

**<end_of_output>**