# resource

## Short summary  
The **resource** package implements a lightweight in‑memory model for managing *ask plans* and their associated tasks.  
It provides:

1. A map abstraction (`askPlanMap`) that stores individual `sonm.AskPlan` objects and can aggregate them into a single `AskPlanResources`.  
2. A pool structure that keeps the master set of resources, tracks used/committed spot‑and‑forward plans, and supports adding/removing plans while keeping the internal state consistent.  
3. A scheduler façade that orchestrates the pool, exposes high‑level operations (consume, release, make room), and converts everything into protobuf‑compatible structures for persistence or communication.

The package is intended to be used by a higher‑level command line tool or service; it can be launched as a normal Go binary (`go run ./cmd/...`) once a `main` package imports it.

---

## Environment variables / flags / cmdline arguments  
| Variable | Purpose |
|----------|---------|
| `LOG_LEVEL` | Optional log level for the zap logger (used in `NewScheduler`). |
| `CONFIG_PATH` | Path to a YAML/JSON file that can be marshalled by `DebugDump`. |

No explicit command line flags are defined inside this package, but the public functions (`NewScheduler`, `ConsumeTask`, etc.) expose all required configuration through their arguments.

---

## Project package structure  

```
resource/
├── ask_plan_map.go
├── plan_pool.go
├── scheduler.go
└── task_pool.go
```

* **ask_plan_map.go** – defines the map alias and two helper methods (`Sum`, `PopLatest`).  
* **plan_pool.go** – implements the core pool logic, including consumption, release, shrinking, committing, and protobuf conversion.  
* **scheduler.go** – thin façade that holds a mutex, a logger, and maps between tasks and ask plans; it delegates to the pool and exposes high‑level operations.  
* **task_pool.go** – manages per‑ask‑plan task resources and provides its own protobuf conversion.

---

## Relations between code entities  

| Entity | Depends on | Notes |
|--------|------------|-------|
| `askPlanMap` | none (type alias) | Used as receiver type for methods in *ask_plan_map.go* and inside the pool struct. |
| `pool` | `askPlanMap`, `zap.SugaredLogger`, `sonm.AskPlanResources` | Holds five maps (`usedSpot`, `usedFw`, etc.) that track plan states; all operations ultimately read/write these maps. |
| `scheduler` | `pool`, `taskPool` | Keeps a mutex, a logger, and two maps: `askPlanPools` (per‑plan task pools) and `taskToAskPlan` (task → plan ID). It forwards calls to the pool and converts everything into protobuf structures. |
| `taskPool` | `sonm.AskPlanResources` | Stores per‑task resources; used by scheduler when a new ask plan is consumed or released. |

The flow of data is:  
1. **NewScheduler** creates a logger, marshals hardware to YAML for debugging, and builds an internal pool (`newPool`).  
2. **ConsumeTask** adds a task to the per‑plan pool and calls `pool.Consume`.  
3. **MakeRoomAndCommit** orchestrates shrinking of spot/forward pools, logs progress, releases the original plan, commits it into the appropriate map, and returns any ejected plans.  
4. **DebugDump** serialises the whole scheduler state to a protobuf‑compatible structure for persistence or debugging.

---

## Edge cases / launch scenarios  

| Scenario | How to launch |
|----------|---------------|
| As a command line tool | `go run ./cmd/main.go` – the main package should import `"github.com/sonm-io/core/resource"` and call `resource.NewScheduler`. |
| As a service | Build binary: `go build -o bin/resource-service ./cmd/...`; then start it with environment variables (`LOG_LEVEL`, `CONFIG_PATH`). |
| With custom logger | Pass an existing zap.SugaredLogger to `NewScheduler` or let the function create one internally. |

---

## Summary of logic  

* **ask_plan_map.go**  
  * Provides a convenient alias for a map keyed by plan ID and two helper methods:  
    - `Sum()` aggregates all plans into a single resource set.  
    - `PopLatest()` removes the most recent plan (by timestamp) from the map.

* **plan_pool.go**  
  * The `pool` struct keeps five maps that track used, committed, and ejected plans.  
  * `newPool(log, resources)` creates an empty pool with all internal maps initialised.  
  * `Consume(planID, plan)` pulls free resources from the master set, verifies availability, and stores the plan in the appropriate map.  
  * `Release(id)` removes a plan from any of the five maps.  
  * Shrinking helpers (`shrinkSpotPool`, `shrinkCommitedSpotPool`) subtract committed forward/spot sums from the master set, add the plan’s resources, and eject plans until required resources are satisfied.  
  * `MakeRoomAndCommit` orchestrates shrinking, logs progress, releases the original plan, commits it into the chosen map, and returns a list of ejected plan IDs.  
  * `ToProto()` deep‑copies the pool state into a protobuf‑compatible structure.

* **scheduler.go**  
  * Holds a mutex (`mu`) to guard concurrent access.  
  * `NewScheduler(hardware, resources)` creates a logger, marshals hardware for debugging, and builds an internal pool.  
  * `DebugDump()` serialises the scheduler state into a protobuf snapshot.  
  * `AskPlanIDByTaskID(taskID)` looks up the plan ID for a given task.  
  * `Consume(planID, resources)` creates a per‑plan task pool (`newTaskPool`) and forwards consumption to the underlying pool.  
  * `MakeRoomAndCommit` calls the pool’s room‑making logic and returns any ejected plans.  
  * `ConsumeTask(taskID, planID, resources)` copies supplied resources into a new `AskPlanResources`, updates internal maps, and forwards consumption to the per‑plan pool.  
  * `Release(planID)` removes a plan from the scheduler’s map, releases it from the underlying pool, and logs completion.  
  * `ReleaseTask(taskID)` frees a task from its associated ask plan.  
  * `ResourceByTask(taskID)` retrieves the resources allocated to a specific task.  
  * `OnDealFinish(taskID)` convenience wrapper that releases a task and cleans up the scheduler’s mapping after a deal finishes.

* **task_pool.go**  
  * The `taskPool` struct holds a master set of resources (`all`) and a map of used tasks.  
  * `newTaskPool(resources)` creates an empty pool with the supplied base resources.  
  * `Consume(ID, resources)` pulls free resources via `pollConsume`, checks for duplicates, and stores the provided resources under that ID.  
  * `Release(ID)` removes a task from the map.  
  * `pollConsume(resources)` calculates free resources (`getFree`) and subtracts the incoming resources to ensure enough capacity.  
  * `getFree()` deep‑copies the master set, computes current usage via `getUsage`, subtracts used amount, and returns the resulting free pool.  
  * `getUsage()` aggregates all consumed tasks into a single resource set.  
  * `ToProto()` converts the internal representation to a protobuf‑compatible type (`sonm.TaskPool`).

All functions are straightforward, with error handling that reports insufficient resources or missing releases. The package is ready for integration into a larger scheduling system where tasks can be added, removed, and queried via the scheduler façade.