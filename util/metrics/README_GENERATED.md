## Package: `metrics`

This package provides a Prometheus exporter for exposing application metrics. It initializes an HTTP server that serves metrics at the `/metrics` endpoint.

**Configuration:**

*   **`addr` (string):** The TCP address to listen on (required).
*   **`zap.SugaredLogger`:** Optional logger dependency. If not provided, a no-op logger is used.

**Files:**

```
util/
└── metrics/
    └── prometheus.go
```

**Functionality:**

1.  **`NewPrometheusExporter(addr string, opts ...Option)`:** Creates a new Prometheus exporter instance. `addr` is the listening address. `opts` allows customization (e.g., logger).
2.  **`Serve(ctx context.Context)`:** Starts the HTTP server. It listens on the configured address and serves metrics at `/metrics`. The server runs until the context is cancelled.
3.  **`newHandler()`:** Creates the HTTP handler for serving Prometheus metrics.

**Relations:**

*   The `PrometheusExporter` struct holds the server and logger.
*   `Serve` uses `net/http` and `promhttp.Handler()` to handle requests.
*   `Option` functions allow for dependency injection (e.g., custom logger).