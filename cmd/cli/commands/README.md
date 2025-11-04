# `<commands>` – CLI package for the *sonmcli* application

## Short overview  
The `commands` package implements a full command‑line interface for managing all SONM entities (blacklist, workers, orders, profiles, tokens, etc.).  It is built on top of Cobra and gRPC, with a central client factory (`client.go`) that creates the various RPC clients used by each subcommand.  The package exposes a root command tree under `cmd/cli` and provides helper functions for printing, error handling, and configuration.

---

## Environment variables / flags  
| Flag | Description | Source file |
|------|--------------|-------------|
| `nodeAddressFlag` | Node endpoint (used by all RPC clients) | `common.go` |
| `outputModeJSON` | Boolean flag to enable JSON output | `common.go` |
| `outputModeFlag` | Name of the output‑mode flag | `common.go` |
| `timeoutFlag` | Timeout value for gRPC calls | `common.go` |
| `keystoreFlag` | Directory where the keystore lives | `common.go` |
| `configFlag` | Path to a config file read by `config.NewConfig` | `common.go` |
| `workerAddressFlag` | Address of the current worker (used in `worker.go`) | `worker.go` |

All flags are registered as persistent flags on the root command (`rootCmd`) and inherited by subcommands.

---

## File structure  
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
```

---

## How the code works together  

1. **Client creation** – `client.go` defines a single helper (`newClientConn`) that builds a gRPC connection from the node address flag and insecure mode flag.  All other constructors (`newWorkerManagementClient`, `newMasterManagementClient`, etc.) reuse this connection to create typed clients for each service.

2. **Root command** – defined in `common.go` as `rootCmd`.  The `init()` function registers all sub‑commands from the various files (e.g., `blacklistRootCmd`, `masterRootCmd`, `orderRootCmd`, etc.) and sets persistent flags that are shared by all commands.

3. **Command hierarchy** – each file defines a root command for its domain (`blacklistRootCmd`, `workerMgmtCmd`, `askPlansRootCmd`, …) and several sub‑commands.  The `init()` in each file adds those sub‑commands to the root, so that running `sonmcli <domain> <subcommand>` dispatches correctly.

4. **Execution flow** – every command follows the same pattern:  
   * Create a timeout context (`newTimeoutContext`).  
   * Build the appropriate client (via constructors in `client.go`).  
   * Perform an RPC call, handle errors, and print results using helper functions from `printers.go`.  

5. **Printing helpers** – `printers.go` contains all formatting logic for the various data types (orders, deals, workers, etc.).  The helpers are used by each command to keep the command files focused on business logic.

6. **Configuration & state** – `common.go` also defines helper functions (`getDefaultKey`, `keystorePath`, `initKeystore`) that load the keystore and provide default keys for RPC calls.  These are used by commands such as `login.go`, `master.go`, etc.

---

## Edge cases / launch scenarios  

| Command | Typical usage | Edge case |
|---------|---------------|-----------|
| `sonmcli blacklist list [addr]` | Show blacklist; optional address overrides default key | If no argument, uses the default key from keystore. |
| `sonmcli master confirm <worker>` | Confirm a worker registration | Must be run after adding a worker via `master add`. |
| `sonmcli order create <bid.yaml>` | Create an order from YAML file | The file must exist; otherwise command fails with “file not found”. |
| `sonmcli token transfer TO AMOUNT` | Transfer tokens between accounts | Requires two arguments; missing one triggers a Cobra error. |

All commands can be invoked directly or via the root command tree, e.g., `sonmcli worker metrics`.  The package also contains unit tests (`*_test.go`) that validate printing and configuration logic.

---

## Summary of relations & potential dead code  

* **Client constructors** are used by all sub‑commands; no unused functions were found.  
* **Helper functions** in `common.go` (e.g., `newTimeoutContext`, `loadKeyStoreWrapper`) are referenced from multiple files, ensuring consistent configuration.  
* The only place where a TODO comment appears is in `worker_benchmarks.go`; the rest of the code seems fully exercised by tests.

---

## Final note  

The package name is inferred as **`commands`** because all source files declare that package and provide a cohesive CLI tree under `cmd/cli`.  All environment variables, flags, and command‑line arguments are listed above for quick reference.