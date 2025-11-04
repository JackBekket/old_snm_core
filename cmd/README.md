# Autocli Package Summary

This package appears to be a collection of command-line tools and utilities, likely related to a distributed computing or resource management system (possibly Sonm, given the `sonmmon` directory). The code is structured around a Cobra-based CLI framework, with numerous subcommands for various operations.

**Configuration:**

*   **`--config` flag:** Required for most commands, specifies the path to a configuration file. Supports `~` for the user's home directory.
*   **Environment variables:** Likely used by underlying dependencies, but not explicitly defined in the provided code.
*   **Configuration files:** The format of these files is not specified, but they likely contain settings for API endpoints, credentials, and other runtime parameters.

**Launch Edgecases:**

*   **Missing `--config`:** Most commands will fail if the `--config` flag is not provided.
*   **Invalid configuration:** Errors in the configuration file will likely cause runtime failures.
*   **Signal handling:** The `WaitInterrupted` function in `cmd/common.go` allows for graceful shutdown on SIGINT/SIGTERM.

**Project Structure:**

```
cmd/
├── autocli/
│   ├── main.go
│   └── proto/
│       └── mod.go
├── cli/
│   ├── commands/
│   │   ├── blacklist.go
│   │   ├── client.go
│   │   ├── common.go
│   │   ├── completion.go
│   │   ├── deals.go
│   │   ├── err_test.go
│   │   ├── login.go
│   │   ├── master.go
│   │   ├── orders.go
│   │   ├── printers.go
│   │   ├── printers_test.go
│   │   ├── profiles.go
│   │   ├── tasks.go
│   │   ├── tokens.go
│   │   ├── version.go
│   │   ├── version_test.go
│   │   ├── worker.go
│   │   ├── worker_askplans.go
│   │   ├── worker_benchmarks.go
│   │   ├── worker_devices.go
│   │   ├── worker_metrics.go
│   │   └── worker_tasks.go
│   ├── config/
│   │   ├── config.go
│   │   └── config_test.go
│   ├── task_config/
│   │   ├── config.go
│   │   ├── config_test.go
│   │   └── load_order.go
│   ├── main.go
├── cobra.go
├── common.go
├── connor/
│   └── main.go
├── dwh/
│   └── main.go
├── lsgpu/
│   └── main.go
├── node/
│   └── main.go
├── optimus/
│   └── main.go
├── oracle/
│   └── main.go
├── pandora/
│   ├── ammo.go
│   ├── ammo_dwh.go
│   ├── ammo_marketplace.go
│   ├── common.go
│   ├── config.go
│   ├── gun.go
│   ├── gun_dwh.go
│   ├── gun_marketplace.go
│   ├── main.go
│   ├── provider.go
│   ├── registry.go
├── qos/
│   └── main.go
├── relay/
│   └── main.go
├── rv/
│   └── main.go
├── secterm/
│   └── main.go
├── sonmmon/
│   ├── TerminusTTFWindows-4.46.0.ttf
│   ├── image.png
│   └── main.go
└── worker/
    └── main.go
```

**Logic Summary:**

The package provides a CLI with numerous subcommands (e.g., `blacklist`, `client`, `deals`, `orders`, `worker`). The `cobra` module handles command parsing and execution. The `config` and `task_config` modules manage application configuration. The `pandora` directory suggests a separate component with its own internal structure (ammo, gun, provider, registry). The other directories (`connor`, `dwh`, `lsgpu`, `node`, `optimus`, `oracle`, `qos`, `relay`, `rv`, `secterm`, `sonmmon`, `worker`) likely represent independent tools or services integrated into the overall system. The `sonmmon` directory contains static assets (TTF font, image).