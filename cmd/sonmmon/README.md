# <end_of_output>  

## Package Overview  
**Name:** `cmd/sonmmon` – a lightweight SDL‑based status monitor for the SONM node.  
The binary reads a YAML config file, connects to an Ethereum node via gRPC, pulls worker metrics, and renders them in a window with a background image.

---

### 1. Configuration & Environment  

| Variable | Purpose |
|----------|---------|
| `SONM_USER` | User name used by `guessHomeViaEnv()` to locate the home directory (`$HOME/.sonm/cli.yaml`). |
| `$DISPLAY` | If unset, an Xorg server is started automatically in `initGraphics()`. |

**Command‑line / flags**  
The binary accepts no explicit flags; it relies on environment variables and a default config path.  
It can be launched as:  

```bash
$ go run cmd/sonmmon/main.go   # during development
$ ./cmd/sonmmon                 # after `go build`
```

---

### 2. File Structure (project package)

```
cmd/
└─ sonmmon/
   ├─ TerminusTTFWindows-4.46.0.ttf      # TTF font for rendering text
   ├─ image.png                           # background image for the SDL window
   └─ main.go                              # Go source – the only code file in this package
```

---

### 3. Core Code Flow  

| Section | Key Functions / Types | What it does |
|---------|-----------------------|---------------|
| **Home & Config** | `guessHomeDir`, `guessConfigPath` | Resolve `$HOME/.sonm/cli.yaml`; fallback to env var or process stats if needed. |
| **Client Setup** | `newClient(ctx, key)` | Builds a TLS‑rotated gRPC client to the local node (`127.0.0.1:15030`). |
| **Worker Status** | `WorkerStatus`, `NewWorkerStatus()`, `(w *WorkerStatus) update(...)` | Holds metrics (uptime, IPs, income, resource percentages). The `update()` method pulls data from the node and calculates derived values. |
| **Graphics Init** | `initGraphics(ctx)` | Starts Xorg if `$DISPLAY` empty; loads font, creates SDL window titled “SONM Status”; scales background image to fit desktop mode. |
| **Main Loop** | `main()` | Sets up logger, config, client, graphics; then repeatedly: <br>• blits background<br>• draws status text (address, master, IP, uptime, income, percentages)<br>• handles keyboard events (alt+F1 to switch TTY, quit on ESC). |

---

### 4. Relations & Dependencies  

* `main()` → `guessHomeDir` → `guessConfigPath` → `config.NewConfig(cfgPath)` → `newClient` → `initGraphics`.  
* The `WorkerStatus.update()` method is called each loop iteration; it uses the gRPC client created in `newClient`.  
* Rendering functions (`drawText`, `Close`) are defined on a `displayCtl` struct that holds SDL surfaces, window, font, and scaling ratios.  
* Font file `TerminusTTFWindows-4.46.0.ttf` is loaded by `ttf.OpenFont`; background image `image.png` is loaded by `img.Load`.  

---

### 5. Edge Cases & Launch Scenarios  

| Scenario | What to watch for |
|----------|--------------------|
| **No `$DISPLAY`** | `initGraphics()` starts an Xorg server automatically; otherwise it just uses the existing display. |
| **Missing config file** | The program logs a fallback path and will error if `$HOME/.sonm/cli.yaml` cannot be read. |
| **Keyboard interrupt** | Pressing ESC ends the main loop; alt+F1 triggers a TTY switch (handled in event loop). |

---

### 6. Summary  

The `cmd/sonmmon` package is a self‑contained SDL monitor that:  
* reads config and Ethereum key,  
* connects to a local node via gRPC,  
* pulls worker metrics into a `WorkerStatus` struct,  
* renders them on a window with a background image, and  
* handles user input for graceful exit.  

All configuration is driven by environment variables (`SONM_USER`, `$DISPLAY`) and the default config path `$HOME/.sonm/cli.yaml`. The only external assets are the TTF font and PNG image located in the same directory as `main.go`.