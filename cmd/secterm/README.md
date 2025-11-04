# secterm

A minimal command‑line tool that reads a YAML configuration, parses an address argument, creates a remote PTY instance and runs it in the background.

---

## Overview  

* **Purpose** – launch a secure PTY session from the CLI.  
* **Entry point** – `cmd/secterm/main.go` (package `main`).  
* **Configuration file** – defaults to `etc/secterm.yaml`; can be overridden with the flag `--config`.  
* **Runtime arguments** – one positional argument: an address string (`ADDR`) that is parsed into an `auth.Addr`.

---

## Configuration & command‑line

| Source | Key | Default / description |
|--------|-----|-----------------------|
| Flag | `--config` | Path to the YAML configuration file (default `"etc/secterm.yaml"`). |
| Argument | `ADDR` | Target address string passed when invoking the binary. |

The tool can be started as:

```bash
$ secterm 192.168.1.10:22
```

or with a custom config path:

```bash
$ secterm --config=./mycfg.yaml 192.168.1.10:22
```

---

## Code structure

### Global state  

```go
var (
    configPath string
)
```
Holds the configuration file path; set by `init()`.

### init()  

Registers a Cobra flag:

```go
rootCmd.Flags().StringVarP(&configPath, "config", "c", "etc/secterm.yaml", "Path to the configuration file")
```

* Flag name: `--config` (short `-c`).  
* Variable: `configPath`.  
* Default value: `"etc/secterm.yaml"`.  

### runSecTerm(v string) error  

Core logic:

1. **Parse address** – `addr, err := auth.ParseAddr(v)` converts the CLI argument into an `auth.Addr`.
2. **Load config** – `cfg, err := secshc.NewRPTYConfig(configPath)` reads the YAML file.
3. **Create PTY** – `tty, err := secshc.NewRemotePTY(cfg)` builds a remote PTY instance from that configuration.
4. **Execute session** – `tty.Run(context.Background(), *addr)` starts the PTY in background context.

Errors are wrapped with a message and returned to the caller.

### rootCmd definition  

```go
var rootCmd = &cobra.Command{
    Use:   "secterm ADDR",
    Short: "Secure PTY",
    Args:  cobra.ExactArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        if err := runSecTerm(args[0]); err != nil {
            fmt.Printf("ERROR: %v\n\r", err)
            os.Exit(1)
        }
    },
}
```

* `Use` – command syntax (`secterm ADDR`).  
* `Short` – brief description.  
* `Args` – expects exactly one argument (the address).  
* `Run` – executes `runSecTerm` with the first argument; on failure prints an error and exits.

### main()  

```go
func main() {
    if err := rootCmd.Execute(); err != nil {
        fmt.Printf("ERROR: %v\n\r", err)
        os.Exit(1)
    }
}
```

Simply runs the Cobra command tree; on failure prints an error and exits.

---

## Project package structure

```
cmd/
└─ secterm/
   └─ main.go
etc/
└─ secterm.yaml
```

* `main.go` is the only source file in this package.  
* The configuration file lives under `etc/`.  

---

## Edge cases & launch options  

| Scenario | How to invoke |
|----------|---------------|
| Default config path | `secterm 192.168.1.10:22` |
| Custom config path | `secterm --config=./mycfg.yaml 192.168.1.10:22` |
| Verbose output (future extension) | Add a flag like `--verbose` to control logging level. |

---

All parts together provide a minimal CLI tool that reads a configuration file, parses an address argument, constructs a remote PTY session and executes it.