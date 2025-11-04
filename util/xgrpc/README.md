# <xgrpc> Package Overview  

**Package name:** `xgrpc` – a small gRPC helper library that bundles client/server construction, authentication, tracing, metrics and a few convenience helpers for method handling.

---

## 1. Short summary of the provided files

| File | Purpose |
|------|---------|
| **client.go** | Implements two public constructors (`NewClient`, `NewQUICClient`) that create a gRPC client connection from an address string, optional TLS credentials and a list of dial options. It also contains a small helper (`newTracer`) for building an OpenTracing tracer used by the client. |
| **client_test.go** | A unit‑test that spins up a local server, serves it in a goroutine, then exercises both secure (TLS) and insecure clients against a dummy method. The test demonstrates how to use `NewClient`/`NewQUICClient`. |
| **credentials.go** | Defines a thin wrapper type around gRPC transport credentials that keeps the TLS configuration handy (`TransportCredentials`). It also provides a constructor (`NewTransportCredentials`) that creates an instance from a `tls.Config`. |
| **method.go** | Provides a small struct (`MethodInfo`) and helper functions to parse a fully‑qualified method path into service/method components. |
| **metrics.go** | Registers a Prometheus gauge metric for active connections, implements a minimal `Handler` type with four methods that tag/handle RPC and connection events. |
| **options.go** | The heart of the package: functional options (`ServerOption`) that mutate an internal `options` struct; helper functions to add credentials, tracing, authentication, logging, rate‑limiting, etc.; and a constructor (`newOptions`) that builds the final option set used by `NewServer`. |
| **server.go** | Exposes two public helpers: `NewServer`, which creates a gRPC server with all chained unary/stream interceptors, and `Services`, which returns a slice of registered service names. |

---

## 2. Environment variables / flags / cmdline arguments

The package itself does not read any environment variable directly; however the following values are expected to be supplied by callers:

| Variable | Meaning |
|----------|---------|
| `addr` (string) | Network address for a client connection (`NewClient`, `NewQUICClient`). |
| `tlsConfig` (*tls.Config) | TLS configuration used by both server and client. |
| `opts ...grpc.DialOption` | Optional dial options passed to the client constructors. |
| `extraOpts ...ServerOption` | Optional server options passed to `NewServer`. |

Typical command‑line flags that a CLI program might use when launching an xgrpc server:

```bash
# Example: run a local test server
go run ./cmd/xgrpc_server -addr="localhost:50051" -tls=true
```

The flag values would be forwarded into the constructors above.

---

## 3. Project package structure

```
util/
└─xgrpc/
   ├─ client.go
   ├─ client_test.go
   ├─ credentials.go
   ├─ method.go
   ├─ metrics.go
   ├─ options.go
   └─ server.go
```

---

## 4. Edge cases for launching

| Scenario | How to launch |
|----------|---------------|
| **As a library** – import `github.com/sonm-io/core/util/xgrpc` in your own code and call `xgrpc.NewServer(...)`. |
| **With TLS credentials** – create a `*tls.Config`, wrap it with `xgrpc.NewTransportCredentials(cfg)` and pass that as the first argument to `NewClient` or `NewQUICClient`. |
| **Using QUIC transport** – supply a non‑nil `tlsConfig` to `NewQUICClient`; the function will dial a QUIC session, wrap it into an `xnet.Conn`, append that connection to the dial options and then call `newClient`. |
| **Running tests** – simply run `go test ./util/xgrpc/...` or `go test -v ./util/xgrpc`. The test file demonstrates both secure and insecure client paths. |

---

## 5. Relations between code entities

* `NewServer` → uses the internal `options` struct built by `newOptions`; this struct contains slices of unary/stream interceptors that are chained in `client.go` via `grpc_middleware`.  
* The tracer created by `newTracer()` is passed to the client dial options as an OpenTracing interceptor; it relies on the `Handler` defined in `metrics.go`.  
* `NewClient` and `NewQUICClient` both call `newClient`, which expects a fully‑qualified address string. That string is parsed into an `authEndpoint` (via `auth.ParseAddr`) that yields a network address (`Addr()`) and an Ethereum address (`ETH()`); the latter is used by the authentication interceptor defined in `options.go`.  
* The helper `ParseMethodInfo(fullMethod)` from `method.go` is used by the rate‑limiter in `options.go`; it splits a method path into service/method components that are later passed to the unary/stream interceptors.  
* The gauge metric `connectionsGauge` defined in `metrics.go` is updated by `HandleConn`, which is invoked automatically when a connection begins or ends thanks to the interceptor chain built in `options.go`.  

Overall, the package offers a clean separation: *client construction*, *server construction*, *method parsing*, *metric handling* and *option assembly*. All pieces are wired together through the functional option pattern (`ServerOption`) and the OpenTracing/Prometheus interceptors.

---

**<end_of_output>**