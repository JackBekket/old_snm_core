# insonmnia/matcher/matcher.go  
# Package `matcher`  
  
## Imports  
```go  
import (  
	"context"  
	"crypto/ecdsa"  
	"errors"  
	"fmt"  
	"math/big"  
	"time"  
  
	"github.com/sonm-io/core/blockchain"  
	"github.com/sonm-io/core/insonmnia/dwh"  
	"github.com/sonm-io/core/proto"  
	"github.com/sonm-io/core/util"  
	"github.com/sonm-io/core/util/multierror"  
	"go.uber.org/zap"  
)  
```  
The package relies on the core blockchain API, a DWH client, protobuf types (`proto`), utility helpers and Uber’s Zap logger.  
  
## External data / input sources  
| Source | Purpose |  
|--------|---------|  
| `sonm.DWHClient` | Fetches matching orders from the distributed wallet hub. |  
| `blockchain.API` | Interacts with the blockchain market (order status, deal opening). |  
| `*ecdsa.PrivateKey` | Signing key used when opening a deal. |  
  
## TODOs  
No explicit `TODO:` comments are present in this file; all functionality is already implemented.  
  
---  
  
# Summary of major code parts  
  
## 1. Configuration structs  
- **YAMLConfig** – lightweight config that can be merged into another component’s YAML.    
  ```go  
  type YAMLConfig struct {  
  	PollDelay  time.Duration `yaml:"poll_delay" default:"30s"`  
  	QueryLimit uint64        `yaml:"query_limit" default:"50"`  
  }  
  ```  
- **Config** – runtime configuration used by the matcher implementation.    
  ```go  
  type Config struct {  
  	Key        *ecdsa.PrivateKey  
  	PollDelay  time.Duration  
  	DWH        sonm.DWHClient  
  	Eth        blockchain.API  
  	QueryLimit uint64  
  	Log        *zap.SugaredLogger  
  }  
  ```  
  The `validate()` method sets defaults, checks required fields and returns a combined error via `multierror`.  
  
## 2. Matcher implementation  
- **type matcher struct** holds a pointer to the config.  
- **NewMatcher(cfg *Config)** validates the config and returns an instance that satisfies the `Matcher` interface.  
  
## 3. Core logic – `CreateDealByOrder`  
```go  
func (m *matcher) CreateDealByOrder(ctx context.Context, order *sonm.Order) (*sonm.Deal, error)  
```  
* Starts a ticker with the configured poll delay.  
* In an infinite loop:  
  1. Wait for either context cancellation or a tick.  
  2. Verify that the target order exists (`checkIfOrderExists`).  
  3. Retrieve matching orders from DWH (`getMatchingOrders`).  
  4. Iterate over each returned order, reorder them into bid/ask pair (`reorderOrders`), open a deal on the blockchain market (`openDeal`) and log progress.  
* Returns the first successfully opened deal.  
  
## 4. Helper methods  
| Method | Purpose |  
|--------|---------|  
| `checkIfOrderExists(ctx context.Context, id *big.Int)` | Queries the blockchain for the order status; ensures it is active. |  
| `getMatchingOrders(ctx context.Context, id *big.Int)` | Calls DWH to fetch orders that match the given ID and converts the reply into a slice of `*sonm.Order`. |  
| `openDeal(ctx context.Context, bid, ask *sonm.Order)` | Opens a deal between two orders via the blockchain API. |  
| `reorderOrders(one, two *sonm.Order)` | Determines which order is a bid and which is an ask; returns them in that order. |  
  
## 5. Disabled matcher  
A minimal stub implementation (`disabledMatcher`) exists for testing or fallback purposes.  
  
---  
  
All functionality is self‑contained within the `matcher` package and can be integrated into larger systems via its exported interface.  
  
# insonmnia/matcher/matcher_test.go  
**Package / Component**    
`matcher`  
  
---  
  
### Imports  
  
| Package | Purpose |  
|---------|---------|  
| `context` | Provides context handling for timeouts and cancellations |  
| `fmt` | Formatting utilities (used in error messages) |  
| `testing` | Test framework integration |  
| `time` | Time duration constants |  
| `github.com/ethereum/go-ethereum/crypto` | Key generation helper |  
| `github.com/golang/mock/gomock` | Mocking support for unit tests |  
| `github.com/sonm-io/core/blockchain` | Blockchain API mock and real client |  
| `github.com/sonm-io/core/proto` | Protobuf definitions (used in mocks) |  
| `github.com/stretchr/testify/assert` | Assertion helpers |  
| `github.com/stretchr/testify/require` | Requirement helpers |  
| `go.uber.org/zap` | Logging helper |  
  
---  
  
### External data / input sources  
  
* **Mocked DWH client** – created by `mockDWH`, returning a set of three orders for a given order type.  
* **Blockchain API mock** – `blockchain.NewMockAPI(ctrl)` and its nested market API mock provide:  
  * `GetOrderInfo` – returns an active order status.  
  * `OpenDeal` – returns a deal ID (or error in failure test).  
* **Configuration struct** – passed to `NewMatcher`, containing key, poll delay, query limit, DWH client, blockchain API and logger.  
  
---  
  
### TODOs  
  
No explicit `TODO:` comments are present in this file.  
  
---  
  
## Summary of major code parts  
  
### 1. `mockDWH`  
  
Creates a mock implementation of the `sonm.DWHClient` interface for unit tests.    
* Builds three orders with IDs 111, 222 and 333, all having the same order type (`t`) and prices equal to their IDs.  
* Sets up an expectation that `GetMatchingOrders` will be called any number of times and returns a reply containing those orders.  
  
### 2. `TestMatcher`  
  
Unit test for the happy‑path creation of a deal by order.    
* Instantiates mocks for blockchain API and market API, sets expectations on `GetOrderInfo` and `OpenDeal`.  
* Calls `NewMatcher` with a configuration that includes the mock DWH client.  
* Creates an order target (ID 1, type BID) and invokes `CreateDealByOrder`.    
* Asserts no error and that a deal object is returned.  
  
### 3. `TestMatcherFailedByTimeout`  
  
Unit test for failure scenario when the context deadline expires before a deal can be created.    
* Similar setup as in `TestMatcher`, but the market API’s `OpenDeal` returns an error.  
* Uses a very short timeout (1 ms) to trigger the failure path.  
* Asserts that an error is returned and matches the expected message `"context deadline exceeded"`.  
  
### 4. `TestMatcherConfigValidate`  
  
Simple test ensuring that `NewMatcher` validates its configuration correctly.    
* Calls `NewMatcher` with a minimal config (zero poll delay, nil key, DWH, Eth) and expects an error to be returned.  
  
---  
  
All tests rely on the helper `mockDWH`, the mock blockchain API, and the logger from `zap`. The file therefore provides both test scaffolding and a small helper for mocking external dependencies.  
  
