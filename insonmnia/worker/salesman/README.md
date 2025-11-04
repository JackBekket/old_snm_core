## Salesman Package Summary

The `salesman` package manages the lifecycle of deals, resource allocation, and blockchain synchronization within the Sonm network. It orchestrates interactions between hardware, cgroups, the Ethereum blockchain, and a matcher service to fulfill compute requests. The package relies heavily on dependency injection via functional options for configuration.

**Configuration:**

*   **Environment Variables:** None explicitly mentioned, but configuration likely loaded from YAML.
*   **Flags/Cmdline Arguments:** Not directly exposed in the provided code.
*   **Files:** `config.yaml` (implied for YAMLConfig).
*   **Paths:** Not explicitly defined, but likely uses standard Sonm configuration paths.

**Launch Edgecases:**

The package is designed to be initialized with dependencies via functional options. Missing dependencies will cause validation errors. The `Run` method starts the main loop, which requires a properly configured `Salesman` instance.

**Project Structure:**

```
insonmnia/worker/salesman/
├── options.go
└── salesman.go
```

**Relationships:**

The `salesman.go` file depends on the `options.go` file for dependency injection. The `Salesman` struct orchestrates interactions between the blockchain, hardware, cgroups, and matcher. The `syncWithBlockchain` and `syncPlanWithBlockchain` functions ensure consistency between the internal state and the Ethereum blockchain.

**Unclear Places/Dead Code:**

The `TODO` comments in `salesman.go` indicate incomplete features: restoring tasks, refactoring network flags, and optimizing deal closing logic. These areas may contain unfinished or suboptimal code.