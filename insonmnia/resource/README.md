# SONM Resource Management Package

This package manages resource allocation for tasks and ask plans within a distributed computing environment (SONM). It provides mechanisms for tracking available resources, consuming them for new requests, releasing them when finished, and querying current usage. The core components include hardware abstraction, task pools, schedulers, and plan management structures.  The system appears designed to handle both spot and forward-based resource allocation models with eviction/preemption capabilities.

## File Structure:

```
insonmnia/resource/
├── ask_plan_map.go       # Manages a map of AskPlans for aggregation & retrieval.
├── hardware.go           # (Not provided in the snippets, but assumed to define OS-level resource access)
├── plan_pool.go          # Tracks available and committed resources for different plans.
├── scheduler.go          # Orchestrates task allocation, consumption, release, and eviction.
└── task_pool.go          # Manages a pool of AskPlanResources by ID (task/plan).
```

## Configuration & Environment Variables:

No explicit environment variables or command-line arguments are present in the provided code snippets. However, resource limits and hardware configurations would likely be set externally during initialization. The `hardware` component (not shown) is expected to handle OS-level settings for CPU, GPU, storage, etc.  The system relies on external inputs like `sonm.AskPlanResources` which could come from a configuration file or API endpoint.

## Edge Cases & Launching:

This package appears designed as an internal component within a larger SONM deployment. It's not directly executable; it requires integration with other services (e.g., task scheduler, resource broker).  The `scheduler.go` suggests that the system is intended to run continuously and respond to external requests for resource allocation/release via API calls or message queues.

## Unclear Places & Dead Code:

*   **`resource/plan_pool.go:TODO: do we need to free it? or only spot?`**: This comment indicates uncertainty about whether resources should be released immediately after committing a plan, potentially leading to resource leaks if not handled correctly.
*   **`resource/scheduler.go://TODO: rework needed — looks like it should not be here`**: Suggests that the `AskPlanIDByTaskID` function may be misplaced or poorly designed and requires refactoring.

## Package Logic Summary:

The package operates by maintaining internal maps of allocated resources (by task ID, ask plan ID) and tracking total availability. The scheduler orchestrates resource consumption/release based on incoming requests, potentially evicting lower-priority plans to make room for new ones.  Deep copying is used extensively to prevent race conditions in concurrent operations. Protocol buffer serialization (`ToProto`) enables communication with external components.