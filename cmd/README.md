# Package `cmd`

The **`cmd`** package is a lightweight wrapper around the Cobra CLI library that builds and runs a command‑line application.  
It exposes an `AppContext` type, helper functions for flag handling, and a `NewCmd(runner Runner)` constructor that turns any `Runner` into a fully‑featured Cobra command.

---

## File structure

```
cmd/
├─ cobra.go
├─ common.go
└─ main.go (implicit – the package’s entry point)
autocli/
│  ├─ main.go
│  └─ proto/mod.go
cli/
│  ├─ commands/blacklist.go
│  ├─ commands/client.go
│  ├─ commands/common.go
│  ├─ commands/completion.go
│  ├─ commands/deals.go
│  ├─ commands/err_test.go
│  ├─ commands/login.go
│  ├─ commands/master.go
│  ├─ commands/orders.go
│  ├─ commands/printers.go
│  ├─ commands/printers_test.go
│  ├─ commands/profiles.go
│  ├─ commands/tasks.go
│  ├─ commands/tokens.go
│  ├─ commands/version.go
│  ├─ commands/version_test.go
│  ├─ commands/worker.go
│  ├─ commands/worker_askplans.go
│  ├─ commands/worker_benchmarks.go
│  ├─ commands/worker_devices.go
│  ├─ commands/worker_metrics.go
│  ├─ commands/worker_tasks.go
│  └─ config/config.go
├─ connor/
│  └─ main.go
├─ dwh/
│  └─ main.go
├─ lsgpu/
│  └─ main.go
├─ node/
│  └─ main.go
├─ optimus/
│  └─ main.go
├─ oracle/
│  └─ main.go
├─ pandora/
│  ├─ ammo.go
│  ├─ ammo_dwh.go
│  ├─ ammo_marketplace.go
│  ├─ common.go
│  ├─ config.go
│  ├─ gun.go
│  ├─ gun_dwh.go
│  ├─ gun_marketplace.go
│  └─ main.go
├─ qos/
│  └─ main.go
├─ relay/
│  └─ main.go
├─ rv/
│  └─ main.go
├─ secterm/
│  └─ main.go
├─ sonmmon/
│  ├─ TerminusTTFWindows-4.46.0.ttf
│  ├─ image.png
│  └─ main.go
└─ worker/
   └─ main.go
```

---

## Environment variables, flags & command‑line arguments

| Source | Variable / Flag | Description |
|--------|-----------------|-------------|
| `os.Args[0]` | `app.Name` | Base of the first CLI argument (used as the command name). |
| `-c`, `--config` | `ConfigPath` | Path to a configuration file; may contain `~`.  The flag is required and persistent. |
| `-v`, `--version` | `showVersion` | Boolean that, when true, prints version information before running the command. |

The application can be configured by editing the file referenced by the `ConfigPath` flag (e.g., `config.yaml`).  
All flags are added to the root Cobra command via `NewCmd`, so they are available globally.

---

## Core logic

### 1. `AppContext`

```go
type AppContext struct {
    Name      string // current application name
    ConfigPath string // path to config file (may contain ~)
    Version   string // version string
}
```

* Holds the runtime context for a command.
* The method `RunWithHomeExpand` expands the `ConfigPath` using `homedir.Expand`, updates the global `app.ConfigPath`, and then executes the supplied runner.

### 2. `NewCmd(runner Runner) *cobra.Command`

* Builds a Cobra command with:
  - **Use**: the application name (`app.Name`).
  - **PreRunE**: prints version info if `showVersion` is true, then checks required flags.
  - **Run**: executes the command by calling `app.RunWithHomeExpand(runner)`.
* Adds persistent flags for configuration path and version flag; marks the config flag as required.

### 3. Helper functions

| Function | Purpose |
|----------|---------|
| `capitalize(s string)` | Upper‑cases first letter of a string (used in help text). |
| `configFlagHelp()` | Returns help text for the config flag. |
| `versionFlagHelp()` | Returns help text for the version flag. |
| `versionString(name, appVersion string)` | Formats a human‑readable version line (`name appVersion (platform)`). |
| `checkRequiredFlags(flags *pflag.FlagSet)` | Validates that all flags annotated with `cobra.BashCompOneRequiredFlag` have been set; returns an error if any are missing. |

All of the above pieces together provide a lightweight wrapper around Cobra for launching a command‑line application, handling configuration path expansion, and displaying version information.

---

## Edge cases & launch scenarios

| Scenario | How to run |
|----------|------------|
| **Local development** | `go run cmd/main.go` – runs the root command with all subcommands (autocli, cli, etc.). |
| **Binary build** | `go build -o bin/cmd ./cmd/...` then execute `./bin/cmd -c config.yaml`. |
| **Using persistent flags** | Flags defined in `NewCmd` are inherited by all child commands; e.g., `--config` can be set once at the root level and used by any subcommand. |
| **Version output** | If `-v` is supplied, the command prints a version line before executing the runner. |

---

## Relations between code entities

* The global variable `app` (of type `AppContext`) is populated in `NewCmd` and later mutated by `RunWithHomeExpand`.  
* The flag set created in `NewCmd` feeds into the Cobra command; the helper functions supply help text that is shown when running with `--help`.  
* The runner passed to `NewCmd` can be any function matching the signature `func(app AppContext) error`; this allows each subcommand (e.g., `autocli/main.go`) to implement its own logic while reusing the same context handling.

---

**<end_of_output>**