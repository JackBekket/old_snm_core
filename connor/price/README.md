# Package `price`

The **`price`** package implements two kinds of mining‑profit providers – a dynamic CoinMarketCap provider (`cmc`) and a simple static provider (`static`).  
It exposes a small set of interfaces that allow the rest of the project to load, validate and update prices in a uniform way.  

## Quick overview

| File | Purpose |
|------|---------|
| `config.go` | Core abstractions: `Provider`, `Updateable`, `Factory`; YAML‑aware `SourceConfig`; factory constructor (`NewFactory`). |
| `cmc_wtm.go` | Concrete implementation of a CoinMarketCap provider; configuration struct, factory, provider type and calculation logic for Ethereum/Monero. |
| `static.go` | Simple static price provider – just reads a single integer from YAML and exposes it as a `big.Int`. |
| `utils.go` | HTTP helpers that fetch token parameters from the *whattomine* API and current USD prices from CoinMarketCap. |
| `cmc_wtm_test.go`, `config_test.go` | Unit tests for calculation logic and YAML parsing. |

The package is intended to be used as a library; it can also be invoked directly via a small CLI wrapper that calls the provider’s `Update()` method.

---

## Environment variables, flags & command‑line arguments

| Variable / Flag | Description |
|------------------|-------------|
| `WHAT_TO_MINE_ID` | Coin ID passed to the CoinMarketCap API (`ethWtmID`, `moneroEtmID`). |
| `CONFIG_TYPE` | YAML key that selects either `"cmc"` or `"static"`. |
| `-config <file>` | (Optional) Path to a YAML file that contains the provider configuration. |

> **Note** – The package itself does not expose any command‑line flags; it is meant to be imported by other packages that may provide a CLI.

---

## Project structure

```
connor/price/
├── cmc_wtm.go
├── cmc_wtm_test.go
├── config.go
├── config_test.go
├── static.go
└── utils.go
```

Each file contains one of the following logical groups:

* **`config.go`** – interface definitions, factory constructor and YAML helpers.
* **`cmc_wtm.go`** – CoinMarketCap provider implementation (configuration, factory, provider struct, calculation functions).
* **`static.go`** – Static provider implementation (configuration, factory, provider struct).
* **`utils.go`** – HTTP/JSON helpers that fetch data from external APIs.
* **Test files** – unit tests for the above groups.

---

## Relations between code entities

| Entity | Where defined | How it is used |
|--------|---------------|-----------------|
| `Provider` interface | `config.go` | Implemented by both `cmcPriceProvider` and `staticProvider`. |
| `Updateable` mixin | `config.go` | Provides the `Interval()` method for providers that need periodic updates. |
| `Factory` interface | `config.go` | Implemented by `CoinMarketCapFactory` (in `cmc_wtm.go`) and `StaticFactory` (`static.go`). |
| `NewFactory(t string)` | `config.go` | Switches on the YAML key `"type"` to return either a CoinMarketCap or static factory. |
| `SourceConfig` struct | `config.go` | Wraps a concrete factory so it can be marshalled/unmarshalled as part of a larger configuration tree. |
| `CoinMarketCapConfig` | `cmc_wtm.go` | Holds URL, interval and coin ID for the CoinMarketCap provider. |
| `StaticProviderConfig` | `static.go` | Holds a single integer price value for the static provider. |
| `NewCMCProvider(cfg *CoinMarketCapConfig, margin float64)` | `cmc_wtm.go` | Factory constructor that creates a `cmcPriceProvider`. |
| `calculateEthPrice`, `calculateXmrPrice` | `cmc_wtm.go` | Calculation functions used by the provider to turn raw token price into per‑hash‑per‑second value. |
| `getTokenParamsFromWTM(id int)` | `utils.go` | HTTP helper that fetches JSON from *whattomine* and returns a `coinParams`. |
| `getPriceFromCMC(url string)` | `utils.go` | HTTP helper that fetches USD price data from CoinMarketCap. |

The tests in `cmc_wtm_test.go` exercise the calculation functions, while `config_test.go` verifies that YAML parsing correctly creates either a `CoinMarketCapFactory` or a `StaticFactory`.

---

## Edge cases for launching

* **As a library** – Import `price` into another package and call:

```go
cfg := &price.CoinMarketCapConfig{URL:"http://...", WhatToMineID:ethWtmID, Interval:3*time.Minute}
prov := price.NewCMCProvider(cfg, 1.0)
prov.Update(context.Background())
fmt.Println(prov.GetPrice())
```

* **As a CLI** – A small wrapper could read a YAML file (`-config` flag), unmarshal it into `SourceConfig`, call `Init()` to get a concrete provider and then invoke its `Update()` method in a loop that sleeps for the configured interval.

---

## Summary of logic

1. **Configuration** – The YAML key `"type"` selects either `"cmc"` or `"static"`.  
   * For `"cmc"` the config contains URL, coin ID and update interval; for `"static"` it contains only a price integer.
2. **Factory construction** – `NewFactory` returns the appropriate factory implementation; each factory implements `Config()`, `ValidateConfig()` and `Init(margin float64)` to create a concrete provider.
3. **Provider execution** –  
   * The CoinMarketCap provider (`cmcPriceProvider`) fetches current USD price via `getPriceFromCMC`, obtains network parameters via `getTokenParamsFromWTM` and then calculates the per‑hash‑per‑second value with either `calculateEthPrice` or `calculateXmrPrice`.  
   * The static provider simply returns the stored integer.
4. **Testing** – Unit tests confirm that calculation functions produce deterministic results for given inputs, and that YAML parsing correctly creates the right factory type.

The package is now ready to be integrated into a larger mining‑profit aggregation system or used directly as a small CLI tool.