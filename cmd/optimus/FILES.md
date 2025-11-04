# cmd/optimus/main.go  
**Package / Component**    
`main`  
  
### Imports  
```go  
import (  
	"context"  
	"fmt"  
  
	"github.com/noxiouz/zapctx/ctxlog"          // logger helper  
	"github.com/sonm-io/core/cmd"               // command‑line framework  
	"github.com/sonm-io/core/insonmnia/version"  // version handling  
	"github.com/sonm-io/core/optimus"           // core Optimus logic  
	"go.uber.org/zap"  
	"go.uber.org/zap/zapcore"  
)  
```  
  
### External data / input sources    
| Source | Description |  
|--------|-------------|  
| `app.ConfigPath` | Path to the configuration file that will be parsed by `optimus.LoadConfig`. |  
| `cfg.Restrictions` | Optional restrictions passed to `optimus.RestrictUsage`. |  
| `cfg.Logging.LogLevel()` | Logging level used when building the Zap logger. |  
| `app.Version` | Version string supplied to the Optimus instance via `optimus.WithVersion`. |  
  
### TODOs  
No explicit `TODO:` comments are present in this file, but future improvements could include:  
- Adding error handling for missing config fields.  
- Enhancing logging configuration (e.g., adding more output paths).  
  
---  
  
## Summary of major code parts  
  
### 1. `main()` – program entry point    
```go  
func main() {  
	cmd.NewCmd(run).Execute()  
}  
```  
Creates a new command using the `cmd` package, passing the `run` function as its handler, and immediately executes it. This is the bootstrap that starts the whole application.  
  
### 2. `run(app cmd.AppContext) error` – core workflow    
The `run` function orchestrates the entire startup sequence:  
  
1. **Configuration loading**    
   ```go  
   cfg, err := optimus.LoadConfig(app.ConfigPath)  
   ```  
   Loads a configuration structure from the path supplied by the command context.  
  
2. **Optional restrictions handling**    
   If the loaded config contains restrictions, they are applied via `optimus.RestrictUsage` and cleaned up with a deferred delete.  
  
3. **Zap logger setup**    
   A `zap.Config` is built with console encoding, development mode off, and a single output path (`stdout`). The level is set from the configuration’s logging level, and the encoder config is tweaked to use capital color levels.  
  
4. **Context creation & version validation**    
   ```go  
   ctx := ctxlog.WithLogger(context.Background(), log)  
   version.ValidateVersion(ctx, version.NewLogObserver(log.Sugar()))  
   ```  
   A new context enriched with a logger is created, and the current application version is validated against the configuration.  
  
5. **Optimus instance creation**    
   ```go  
   bot, err := optimus.NewOptimus(cfg,  
       optimus.WithVersion(app.Version),  
       optimus.WithLog(log.Sugar()))  
   ```  
   An `optimus` object (the core business logic) is instantiated with the loaded config and options for versioning and logging.  
  
6. **Execution**    
   ```go  
   return bot.Run(ctx)  
   ```  
   Finally, the Optimus instance runs its main routine using the prepared context.  
  
---  
  
This file serves as the bootstrap of a command‑line tool that loads configuration, sets up structured logging with Zap, validates the application version, and hands control over to an `optimus` core component.  
  
