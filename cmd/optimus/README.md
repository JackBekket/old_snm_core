# Package main (cmd/optimus)

## Short summary  
`main.go` is the bootstrap for a command‑line tool that loads an Optimus configuration file, builds a Zap logger, validates the application version and hands control to the core `optimus` logic. It parses command‑line arguments via the `github.com/sonm-io/core/cmd` framework, then executes the main routine.

## Environment variables / flags / cmd‑line arguments  
| Variable / flag | Description | Default / source |
|------------------|-------------|-------------------|
| `app.ConfigPath` | Path to the configuration file that will be parsed by `optimus.LoadConfig`. | Provided by the command framework (`cmd.NewCmd`). |
| `cfg.Restrictions` | Optional restrictions passed to `optimus.RestrictUsage`. | Loaded from the config file. |
| `cfg.Logging.LogLevel()` | Logging level used when building the Zap logger. | From the loaded config. |
| `app.Version` | Version string supplied to the Optimus instance via `optimus.WithVersion`. | Provided by the command framework. |

## File structure  
```
cmd/
└─ optimus/
   └─ main.go
```

## How the application is launched (edge cases)  

1. **Direct execution** – `go run ./cmd/optimus/main.go` will invoke `main()` which immediately executes the command created by `cmd.NewCmd(run).Execute()`.  
2. **Binary build** – `go build -o optimus ./cmd/optimus/main.go` followed by `./optimus --config=... --version=...` will start the same flow, with flags parsed by the `cmd` package.  
3. **Environment‑driven config** – If an environment variable (e.g., `OPTIMUS_CONFIG`) is defined, it can be read into `app.ConfigPath` before execution.

## Relations between code entities  

- `main()` creates a command via `cmd.NewCmd(run)`; the handler function `run` receives a context (`app cmd.AppContext`).  
- Inside `run`, the configuration file path (`app.ConfigPath`) is used by `optimus.LoadConfig`.  
- The loaded config (`cfg`) supplies logging level and optional restrictions that are applied with `optimus.RestrictUsage`.  
- A Zap logger is built, wrapped into a context via `ctxlog.WithLogger`, then validated against the application version.  
- Finally, an Optimus instance (`bot`) is created with options `WithVersion` and `WithLog`, and its `Run(ctx)` method is called to finish execution.

No obvious dead code or missing pieces were detected; all imports are used directly in the flow.