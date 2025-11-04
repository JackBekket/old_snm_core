# cmd/sonmmon/main.go  
# Package / Component    
**Name:** `main` (Linux build)    
  
## Imports  
```go  
import (  
	"context"  
	"crypto/ecdsa"  
	"fmt"  
	"math"  
	"math/big"  
	"os"  
	"os/exec"  
	"os/user"  
	"path"  
	"strconv"  
	"strings"  
	"syscall"  
	"time"  
  
	"github.com/ethereum/go-ethereum/crypto"  
	"github.com/sonm-io/core/cmd"  
	"github.com/sonm-io/core/cmd/cli/config"  
	"github.com/sonm-io/core/insonmnia/auth"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util"  
	"github.com/sonm-io/core/util/netutil"  
	"github.com/veandco/go-sdl2/img"  
	"github.com/veandco/go-sdl2/sdl"  
	"github.com/veandco/go-sdl2/ttf"  
	"go.uber.org/zap"  
	"golang.org/x/sync/errgroup"  
	"google.golang.org/grpc"  
	"google.golang.org/grpc/metadata"  
)  
```  
  
## External Data / Input Sources  
| Source | Description |  
|--------|-------------|  
| `os.Getenv("SONM_USER")` | User name for home directory lookup |  
| `/proc/<pid>/cmdline` | Process ID of `sonmnode` to infer user home |  
| `config.NewConfig(cfgPath)` | YAML config file (default: `$HOME/.sonm/cli.yaml`) |  
| `cfg.Eth.LoadKey()` | Ethereum key from config |  
| `netutil.GetPublicIPs()` / `GetAvailableIPs()` | Public and local IP addresses |  
| `image.png` | Background image for the SDL window |  
  
## TODO List  
- **Main loop** – “maybe move to goroutine and wait for signal asynchronously?” (inside event handling)  
  
## Summary of Major Code Parts  
  
### 1. Home & Config Path Helpers    
* `guessHomeDir(log)` – Tries environment variable first, then process stats; returns home directory string or error.    
* `guessHomeViaEnv()` – Reads `SONM_USER` env var and looks up the user via `os/user`.    
* `guessHomeViaProc()` – Executes `pgrep -x -o sonmnode`, parses PID, reads `/proc/<pid>/cmdline`, obtains UID, then resolves home directory.    
* `guessConfigPath(log)` – Builds full path to config file (`$HOME/.sonm/cli.yaml`) and logs fallback if needed.  
  
### 2. Worker Status Model & Update Logic    
* `WorkerStatus` struct holds status flags, timestamps, network metrics, and a nested `Sold` sub‑struct for resource percentages.    
* `NewWorkerStatus()` – Returns an initialized instance with defaults.    
* `(w *WorkerStatus) update(ctx, cc, addr)` – Uses gRPC client to fetch worker status, ask plans, device stats; calculates uptime string, public IP, income per hour, and percentage usage of each resource type.  
  
### 3. Client Connection Setup    
* `newClient(ctx, key)` – Builds a TLS config via `util.NewHitlessCertRotator`, creates wallet authenticator, then returns an gRPC client connection to the local node (`127.0.0.1:15030`).  
  
### 4. Display Control & Rendering    
* `displayCtl` struct holds SDL surfaces, window, font, and scaling ratios.    
* `drawText(x,y,text,color)` – Renders UTF‑8 text onto the surface with optional offset handling.    
* `Close()` – Cleans up SDL resources, Xorg process, fonts, and quits SDL/TTF.    
  
### 5. Graphics Initialization (`initGraphics`)    
* Starts an Xorg server if `$DISPLAY` is empty.    
* Initializes SDL video/events, loads font “TerminusTTFWindows‑4.46.0.ttf”, creates a window titled “SONM Status”.    
* Calculates scaling ratios between background image and desktop mode to center the image.    
* Loads `image.png` as background surface.  
  
### 6. Main Application Flow (`main`)    
1. Creates zap logger, logs start message.    
2. Determines config path, loads config, reads Ethereum key.    
3. Builds gRPC client connection via `newClient`.    
4. Initializes graphics with `initGraphics`; defers cleanup.    
5. Starts an errgroup goroutine to wait for interruption (`cmd.WaitInterrupted`).    
6. Enters a loop that:  
   * Blits background image onto the surface, updates window.  
   * Draws status text (worker address, master, IP, uptime, income, resource percentages).    
   * Handles keyboard events: alt+F1 triggers TTY switch; quit event ends loop.    
   * Calls `worker.update` to refresh metrics each iteration.    
7. Waits for goroutine completion and logs exit.  
  
All major parts are now summarized for package‑level documentation.  
  
