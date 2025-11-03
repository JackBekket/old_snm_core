# util/metrics/prometheus.go  
## Package: `metrics` Summary  
  
This package provides a Prometheus exporter for application metrics, serving them via an HTTP endpoint at `/metrics`. It uses the `prometheus/client_golang` library to expose metrics and allows customization through functional options.  
  
**Imports:**  
  
*   `context`: For managing server lifecycle with context cancellation.  
*   `net`: For creating TCP listeners.  
*   `net/http`: For serving HTTP requests.  
*   `github.com/prometheus/client_golang/prometheus/promhttp`: Prometheus HTTP handler for `/metrics`.  
*   `go.uber.org/zap`: Structured logging library (optional).  
*   `golang.org/x/sync/errgroup`:  For managing concurrent goroutines and error handling.  
  
**External Data / Input Sources:**  
  
*   `addr` (string): The TCP address to bind the Prometheus exporter server to (e.g., `:9090`). This is a required input when creating `PrometheusExporter`.  
*   Context (`context.Context`): Used for graceful shutdown of the HTTP server.  
  
**Functional Options:**  
  
The package uses functional options pattern via the `Option` type and functions like `WithLogging` to configure the exporter, specifically allowing injection of a custom `zap.SugaredLogger`. If no logger is provided, it defaults to a no-op logger.  
  
**Major Code Parts Summary:**  
  
*   **Options Struct & Functions (`newOptions`, `Option`, `WithLogging`):**  Handles configuration using functional options for dependency injection (logging).  
*   **PrometheusExporter Struct & New Function:** Defines the exporter struct with address and logger, providing a constructor function to create instances.  
*   **Serve Method:** Starts an HTTP server listening on the specified address, serving Prometheus metrics at `/metrics`. It uses `golang.org/x/sync/errgroup` for managing concurrent operations (server serve) and graceful shutdown based on context cancellation.  Logs start and stop events using injected logger.  
*   **newHandler Function:** Creates an HTTP handler that serves the Prometheus metrics endpoint (`/metrics`) via `promhttp.Handler()`.  
  
**TODOs:**  
  
There are no explicit TODO comments in this code snippet.  
  
