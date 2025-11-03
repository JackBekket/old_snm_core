# xnet Package Summary

The `xnet` package provides utilities for creating network listeners, handling QUIC connections, and resolving external/loopback IP addresses. It supports TCP, UDP, and QUIC protocols with configurable backpressure mechanisms for connection acceptance. The package relies on external services (e.g., AWS checkip endpoint) for public IP resolution and uses caching to optimize performance.

## Project Package Structure:

```
util/
└── xnet/
    ├── listener.go
    ├── quic.go
    └── resolve.go
```

**Configuration:**

*   **`ListenLoopback`, `ListenPacketLoopback`**: Network type (`tcp`, `udp`) and port number (uint16) are required for creating loopback listeners. The external function `LookupLoopbackIP()` is used to obtain loopback IP addresses, which may depend on system configuration.
*   **`ListenQUIC`**: Requires a TLS configuration (`tls.Config`) and QUIC configuration (`quic.Config`). Default QUIC configurations are provided via the `DefaultQUICConfig` function. The network address (string) is also required for binding the listener.
*   **`NewExternalPublicIPResolver`**: Allows configuring the HTTP endpoint used to resolve external public IP addresses. Defaults to "http://checkip.amazonaws.com/". Cache refresh duration can be adjusted implicitly through the resolver's internal timer.

**Edge Cases:**

*   The `ListenLoopback` and `ListenPacketLoopback` functions handle errors during listener creation by closing any partially created listeners before returning an error.
*   `ListenQUIC` handles peer-gone errors (disconnections) gracefully, skipping them while continuing to accept new connections. Other errors will cause the function to return.
*   The external IP resolver caches results for a configurable duration; stale IPs may be returned if the cache hasn't been refreshed.

**Unclear Places/Dead Code:** The `LookupLoopbackIP()` function is called in multiple places but not defined within this package, implying it relies on an external dependency or another part of the codebase. This could introduce hidden dependencies and potential failure points. No dead code was found during review.