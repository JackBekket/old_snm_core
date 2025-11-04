# util

## Overview  
The `util` package bundles a collection of helpers that are used throughout the project: TLS‑certificate rotation, gRPC credential handling, metadata forwarding, a lightweight ticker, and a handful of configuration utilities.  The most visible pieces are:

* **Certificate rotation** – `certs.go` implements an *hitless* cert rotator that periodically generates new X.509 certificates signed with an ECDSA key and exposes them to gRPC via a custom `tls.Config`.
* **gRPC integration** – `grpcsecure.go` wraps the standard gRPC transport credentials so that the generated certificates can be verified on both client and server sides.
* **Metadata forwarding** – `metadata.go` provides a tiny helper that copies incoming request metadata into an outgoing context, useful for proxying or chaining services.
* **Ticker** – `ticker.go` offers an immediate‑tick channel that can be used by any component that needs to fire events right away (e.g. the cert rotator).
* **Config helpers** – `util.go` contains a few functions that build user‑specific directories, format big integers and convert hex strings into Ethereum addresses.

The package is intentionally lightweight; it does not expose a command line interface itself but can be used by a main program or another library (e.g. the `xgrpc` subpackage).

---

## Project structure

```
util/
├─ action/action.go
├─ action/queue.go
├─ certs.go
├─ certs_test.go
├─ config/config.go
├─ config/retag.go
├─ config/retag_test.go
├─ datasize/datasize.go
├─ datasize/datasize_test.go
├─ debug/pprof.go
├─ defergroup/mod.go
├─ grpc.go
├─ grpcsecure.go
├─ grpcutil_test.go
├─ metadata.go
├─ metrics/prometheus.go
├─ multierror/error.go
├─ netutil/net.go
├─ netutil/net_test.go
├─ rest/aes.go
├─ rest/errors.go
├─ rest/options.go
├─ rest/server.go
├─ ticker.go
├─ util.go
├─ util_test.go
├─ xcode/cmd.go
├─ xconcurrency/cncurrency.go
├─ xdocker/reference.go
├─ xdocker/reference_test.go
├─ xdocker/xdocker.go
├─ xdocker/xdocker_test.go
├─ xgrpc/client.go
├─ xgrpc/client_test.go
├─ xgrpc/credentials.go
├─ xgrpc/method.go
├─ xgrpc/metrics.go
├─ xgrpc/options.go
├─ xgrpc/server.go
├─ xnet/listener.go
├─ xnet/quic.go
└─ xnet/resolve.go
```

---

## Environment variables, flags and command‑line arguments

| Variable / flag | Purpose | Default / usage |
|------------------|---------|-----------------|
| `WorkerAddressHeader` (in `grpc.go`) | HTTP header key for worker Ethereum addresses | `"x-worker-eth-addr"` – used by the gRPC client/server to carry an address in the request metadata. |
| `certValidPeriod` (in `certs.go`) | Validity of a generated TLS cert | Default 4 h; can be overridden by passing a different value to `NewHitlessCertRotator`. |
| `--config-dir` (implied) | Path for user‑specific config files | Built by `GetDefaultConfigDir()` in `util.go`; the default is `$HOME/.sonm`. |

The package itself does not expose a command line interface, but a main program can start it with:

```bash
go run ./cmd/main --config-dir=$HOME/.sonm
```

or simply import `util` and call its exported constructors.

---

## Edge cases for launching

1. **Default launch** – Call `NewHitlessCertRotator(ctx, ethPriv)` to obtain a rotator and a TLS config; pass the config to `grpcsecure.NewTLS(cfg)` when creating a gRPC server or client.
2. **Custom validity period** – Use `newHitlessCertRotator(ctx, ethPriv, 6*time.Second)` (as in the test) to shorten the rotation cycle for quick testing.
3. **Immediate ticker** – The `ImmediateTicker` created by `NewImmediateTicker(d)` can be used as a dependency of the cert rotator; it guarantees that the first tick is delivered instantly, so the rotator starts without waiting for the first timer tick.

---

## Code relationships and key entities

| File | Key type / function | Relation |
|------|---------------------|----------|
| `certs.go` | `HitlessCertRotator`, `GenerateCert`, `rotateOnce`, `rotation` | Core TLS‑certificate lifecycle; exposes a `tls.Config` that is consumed by `grpcsecure.NewTLS`. |
| `grpcsecure.go` | `tlsVerifier`, `NewTLS` | Wraps the standard gRPC transport credentials; verifies certificates on both sides. |
| `metadata.go` | `ForwardMetadata` | Simple helper to copy incoming metadata into an outgoing context – used by any gRPC proxy or chain. |
| `ticker.go` | `ImmediateTicker` | Provides a channel that can be consumed by the cert rotator for scheduling. |
| `util.go` | `GetDefaultConfigDir`, `GetDefaultKeyStoreDir`, `StringToEtherPrice`, etc. | Builds paths and formats values used by other subpackages (e.g. `config/retag.go`). |

The flow is:

1. **Main program** creates an ECDSA key (`ethcrypto.GenerateKey()`).
2. Calls `newHitlessCertRotator(ctx, priv, certValidPeriod)` → returns a rotator and a TLS config.
3. Passes the TLS config to `grpcsecure.NewTLS(cfg)` when starting a gRPC server or client.
4. The rotator’s `rotation` goroutine uses an `ImmediateTicker` (created elsewhere) to schedule certificate refreshes; each tick triggers `rotateOnce`, which calls `GenerateCert` and updates the internal cert atomically.
5. The generated certificates are exposed via the TLS callbacks (`GetCertificate`, `GetClientCertificate`) that satisfy the gRPC transport interface.

---

## Summary of what the package does

* **TLS certificate rotation** – Generates X.509 certificates signed with an ECDSA key, stores them in a thread‑safe struct, and refreshes them automatically.
* **gRPC credential handling** – Wraps standard credentials to verify peer certificates and extract Ethereum addresses from metadata.
* **Metadata forwarding** – Copies request metadata into outgoing contexts for gRPC chaining.
* **Immediate ticker** – Provides an instant tick channel that can be used by any component needing a first event immediately (e.g. the cert rotator).
* **Config helpers** – Builds user‑specific directories, formats big integers and converts hex strings to Ethereum addresses.

The package is ready to be imported by higher‑level code; it exposes all necessary constructors (`NewHitlessCertRotator`, `NewTLS`) and a small set of constants that can be referenced elsewhere (e.g. `WorkerAddressHeader`).