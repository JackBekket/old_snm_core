# util/metrics/prometheus.go  
## Package: `metrics`  
  
**Imports:**  
  
*   `context`: For managing the lifecycle of the Prometheus exporter server.  
*   `net`: For creating TCP listeners.  
*   `net/http`: For serving the Prometheus metrics endpoint.  
*   `github.com/prometheus/client_golang/prometheus/promhttp`: For handling Prometheus metrics requests.  
*   `go.uber.org/zap`: For structured logging.  
*   `golang.org/x/sync/errgroup`: For managing concurrent goroutines and error handling.  
  
**External Data/Input Sources:**  
  
*   `addr` (string): The TCP address on which the Prometheus exporter will listen. This is a required input when creating a new exporter.  
*   `context.Context`: Used to control the lifecycle of the server. The `Serve` method blocks until the context is cancelled.  
*   `zap.SugaredLogger`: Optional dependency for structured logging. If not provided, a no-op logger is used.  
  
**TODOs:**  
  
*   None found in the provided code snippet.  
  
**Code Summary:**  
  
### Options Configuration  
  
The `options` struct and associated `Option` function type provide a flexible way to configure the `PrometheusExporter`. This allows for dependency injection, specifically for the logger. The `newOptions` function creates a default configuration with a no-op logger, which can be overridden using the `WithLogging` option.  
  
### Prometheus Exporter Initialization  
  
The `NewPrometheusExporter` function creates a new instance of the `PrometheusExporter`. It takes the listening address (`addr`) as a required argument and accepts optional `Option` functions to customize the exporter's behavior, such as providing a custom logger.  
  
### Metrics Server Lifecycle  
  
The `Serve` method starts the Prometheus exporter HTTP server. It listens on the specified TCP address, handles `/metrics` requests using `promhttp.Handler()`, and logs server start/stop events. The server runs until the provided context is cancelled or an error occurs. The `errgroup` is used to manage the server goroutine and ensure proper shutdown.  
  
### Metrics Handler  
  
The `newHandler` function creates an HTTP handler that serves Prometheus metrics at the `/metrics` endpoint. It uses `promhttp.Handler()` to handle the requests.  
  
