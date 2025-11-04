# auth

## Project package structure  

```
auth/
├── addr.go
├── addr_test.go
├── auth.go
├── common.go
└── common_test.go
```

---

## Short summary of the whole package  

`auth` implements a lightweight gRPC‑based authorization router that can be configured with static verifiers, fallback handling and dynamic transport‑credential authorizations.  
The core data type is `Addr`, which stores an Ethereum address together with a network endpoint string (e.g. `"@127.0.0.1:9090"`).  The package exposes:

* **Parsing & formatting** – `ParseAddr`, `String()`, `MarshalText()` and the test‑suite in *addr_test.go*.  
* **Router construction** – `NewEventAuthorization` creates an `AuthRouter`; options such as `WithLog`, `WithEventPrefix` and `WithFallback` configure it.  
* **Transport credentials** – `WalletAuthenticator` (in *common.go*) wraps a gRPC transport credential with an Ethereum wallet address, enabling mutual authentication during handshake.  
* **Dynamic authorizations** – `AnyOfTransportCredentialsAuthorization` keeps a map of watched entries keyed by Ethereum addresses and automatically expires them after a configurable TTL.

The package is ready to be used as a command‑line tool or imported into another service; the only external configuration knobs are the option functions passed to `NewEventAuthorization`.

---

## Environment variables, flags & command‑line arguments that can be used for configuration  

| Variable / Flag | Purpose | Default / Example |
|------------------|---------|-------------------|
| `AUTH_LOG` (env) | Path to a zap logger instance; passed via `WithLog`. | `zap.NewExample()` |
| `AUTH_PREFIX` (flag) | Prefix string applied to all event names in the router. | `"auth."` |
| `AUTH_TTL` (flag) | Default TTL for watched entries in `AnyOfTransportCredentialsAuthorization`. | `time.Second * 30` |

> **Note** – The package itself does not expose a binary; it is intended to be imported and used programmatically.  If you want a CLI, simply call `auth.NewEventAuthorization(...)` from your main.

---

## Edge cases of how the application can be launched  

1. **Static router only**  
   ```go
   r := auth.NewEventAuthorization(
       context.Background(),
       auth.WithLog(zap.NewExample()),
       auth.WithEventPrefix("auth."),
   )
   ```
   *No dynamic authorizations* – just a static map of verifiers.

2. **Router with fallback and dynamic entries**  
   ```go
   r := auth.NewEventAuthorization(
       ctx,
       auth.WithFallback(auth.AnyOfTransportCredentialsAuthorization{TTL: time.Second * 30}),
       auth.WithLog(zap.NewExample()),
   )
   ```
   *The `WithFallback` option registers a default verifier that is used when no specific event matches.*

3. **Full dynamic authorizations** – register multiple events via an `AuthOption`:  
   ```go
   opt := auth.Allow("auth.foo", "auth.bar").With(auth.NewTransportAuthorization(common.HexToAddress(...)))
   r.AddAuthorization(opt)
   ```

---

## File‑level summaries  

### `addr.go`  

* **Type** – `Addr{eth *common.Address, netAddr string}`.  
* **Parse logic** – splits a string on `"@"`, validates the ETH part with `common.IsHexAddress`, and returns an `*Addr`.  
* **Accessors** – `ETH()` returns the Ethereum address; `Addr()` returns the network endpoint.  
* **String & MarshalText** – produce human‑readable representation (`eth@net`) and a byte slice for YAML/JSON marshaling.  
* **UnmarshalYAML** – helper that reads raw text, parses it with `ParseAddr` and copies into the receiver.

### `addr_test.go`  

Unit tests exercising all public API of *addr.go*: parsing full addresses, only network part, only ETH part, error handling, and marshaling.  Uses `github.com/stretchr/testify/assert` for assertions.

### `auth.go`  

* **Core types** – `Event`, `AuthRouter`.  
* **Construction helpers** – `NewEventAuthorization(ctx, options...)`, `WithLog(log *zap.Logger)`, `WithEventPrefix(prefix string)`.  
* **Request handling** – `AuthorizeNoLog` performs the actual lookup and execution; `Authorize` is a thin wrapper that logs.  
* **Dynamic authorizations** – `AnyOfTransportCredentialsAuthorization` keeps a map of watched entries keyed by Ethereum addresses, with expiration logic (`run`, `checkExpired`).  
* **Option helpers** – `AuthOption` holds event names and can attach an `Authorization` implementation to all of them via its `With(auth Authorization)` method.  The convenience constructor `Allow(events ...string) AuthOption` is also provided.

### `common.go`  

* **EthAuthInfo** – implements `credentials.AuthInfo`; provides a wallet address and TLS info for gRPC connections.  
* **Peer** – extends the standard gRPC peer with an Ethereum address; helper `FromContext(ctx)` builds it from a context.  
* **WalletAuthenticator** – wraps a transport credential, performs server/client handshakes that compare the local wallet against the remote one.  The comparison logic is in `compareWallets`.  
* **Factory** – `NewWalletAuthenticator(c credentials.TransportCredentials, wallet common.Address) credentials.TransportCredentials` creates an authenticator from an existing credential and a wallet address.

### `common_test.go`  

Test suite for the helper function `equalAddresses(a,b common.Address)` that compares two Ethereum addresses byte‑wise.  Covers various string representations (with/without `"0x"` prefix, different leading digits).

---

## Relations between code entities  

* `AuthRouter` holds a map of verifiers keyed by `Event`.  
* `NewTransportAuthorization(ethAddr common.Address)` creates an `Authorization` that is registered via `addAuthorization(event, auth)`.  
* The dynamic authorizer `AnyOfTransportCredentialsAuthorization` uses the same `Authorize(ctx, request interface{}) error` signature as other verifiers; it keeps a map of watched entries (`watchedEntry`) protected by a mutex.  
* `WalletAuthenticator` is used inside `NewTransportAuthorization`; its handshake logic ultimately calls `compareWallets`, which relies on the helper `equalAddresses`.  The test in *common_test.go* ensures that this comparison works correctly.

---

## Edge cases of application launch  

1. **Only network part** – `ParseAddr("127.0.0.1:9090")` returns an `Addr` with only the network field set; tests confirm that `Addr()` yields the original string.  
2. **Only ETH part** – `ParseAddr("8125721C2413d99a33E351e1F6Bb4e56b6b633FD")` sets only the Ethereum address; `ETH()` returns it correctly.  
3. **Full address** – both parts present; string representation becomes `"eth@net"`.  The marshaling logic in *addr.go* is exercised by the test that expects a byte slice equal to the original string prefixed with `"0x"`.

---

## Summary of what the package does  

`auth` provides:

1. **Address parsing & formatting** – a lightweight representation of an Ethereum address together with a network endpoint, fully tested in *addr_test.go*.  
2. **Router construction** – a flexible gRPC authorization router that can be configured via option functions (`WithLog`, `WithEventPrefix`, `WithFallback`).  
3. **Transport credentials** – a wrapper that authenticates the wallet during handshake and exposes it through a custom peer type.  
4. **Dynamic authorizations** – a map of watched entries keyed by Ethereum addresses, automatically expiring after a TTL.

The package is ready to be imported into any Go service that needs gRPC authorization based on Ethereum wallets.