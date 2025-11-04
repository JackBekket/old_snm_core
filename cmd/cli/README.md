# Package **main**

## Short summary  
The `cmd/cli/main` package implements a command‑line interface for the *sonm-io/core* application.  
It defines a root command that aggregates a large set of sub‑commands (blacklist, client, common, completion, deals, err_test, login, master, orders, printers, profiles, tasks, tokens, version, worker and its sub‑tasks).  The package also contains configuration helpers (`config/config.go`, `task_config/config.go`) and test files for both.  
`main.go` bootstraps the CLI by creating a root command with the current application version, executing it, printing any error that occurs, and exiting with status 1 on failure.

## Environment variables / flags / cmd‑line arguments  

| Variable / Flag | Purpose | Source |
|------------------|---------|--------|
| `version.Version` (from `github.com/sonm-io/core/insonmnia/version`) | Provides the current build version to the root command | `config/config.go` |
| `commands.Root()` | Creates the top‑level CLI command tree | `cmd/cli/main.go` |
| `root.Execute()` | Runs all sub‑commands, parses arguments and flags | `main.go` |

The package does not expose any explicit environment variables or custom flags in the provided snippet; however, each of the individual command files (e.g. `blacklist.go`, `client.go`, …) likely defines its own options and arguments.

## Edge cases for launching  

* **Normal launch** – `go run cmd/cli/main.go` will build the binary and execute the root command tree.  
* **Error handling** – if any sub‑command fails, `commands.ShowError(root, err.Error(), nil)` prints a message and the program exits with status 1.  
* **Configuration files** – The package can be configured via `config/config.go` (general config) or `task_config/config.go` (task‑specific config).  These files are automatically loaded by the command infrastructure when the root command is executed.

## Project package structure  

```
cmd/cli/
├── commands/
│   ├── blacklist.go
│   ├── client.go
│   ├── common.go
│   ├── completion.go
│   ├── deals.go
│   ├── err_test.go
│   ├── login.go
│   ├── master.go
│   ├── orders.go
│   ├── printers.go
│   ├── printers_test.go
│   ├── profiles.go
│   ├── tasks.go
│   ├── tokens.go
│   ├── version.go
│   ├── version_test.go
│   ├── worker.go
│   ├── worker_askplans.go
│   ├── worker_benchmarks.go
│   ├── worker_devices.go
│   ├── worker_metrics.go
│   └── worker_tasks.go
├── config/
│   ├── config.go
│   └── config_test.go
├── main.go
└── task_config/
    ├── config.go
    ├── config_test.go
    └── load_order.go
```

## Relations between code entities  

* `main.go` creates the root command and delegates execution to it.  
* The `commands/` package contains all sub‑command implementations; each file defines a distinct CLI action (e.g., `orders.go` handles order management, `worker_*` files handle worker‑related tasks).  These are registered with the root command via `commands.Root()`.  
* Configuration is split into two layers: general settings (`config/config.go`) and task‑specific settings (`task_config/config.go`).  The latter can be loaded by any of the worker sub‑commands.  
* Test files (`*_test.go`) provide unit tests for their corresponding commands, ensuring that each command behaves as expected.

The package therefore provides a fully functional CLI with modular commands, configuration support, and test coverage.