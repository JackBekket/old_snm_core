# util/xnet/listener.go  
## xnet Package Component Summary  
  
**Package Name:** `xnet`  
  
**Imports:**  
*   `fmt`: For formatted I/O.  
*   `net`: Core networking primitives.  
*   `strconv`: String conversion utilities (e.g., port to string).  
*   `time`: Time-related functions, including sleep intervals.  
*   `go.uber.org/zap`: Structured logging library.  
  
**External Data / Input Sources:**  
*   Network type (`tcp`, `tcp4`, `tcp6`, `udp`, `udp4`, `udp6`) as a string input to `ListenLoopback` and `ListenPacketLoopback`.  
*   Port number (uint16) as an input to both functions.  
*   Loopback IP addresses obtained from the `LookupLoopbackIP()` function (not defined in this file, assumed external).  
  
**TODOs:** None found in provided code snippet.  
  
---  
  
### BackPressureListener   
  
This struct wraps a standard `net.Listener` and adds logging via `zap`. The primary functionality is within the overridden `Accept()` method. It implements exponential backoff when encountering temporary network errors during connection acceptance. If an error occurs, it logs the failure with a duration indicating the sleep interval before retrying.  The sleep interval increases exponentially up to a maximum value (`maxSleepInterval`).  
  
---  
  
### ListenLoopback   
  
This function creates and returns a slice of TCP listeners bound to loopback IP addresses on the specified port. It validates that the provided network type is one of `tcp`, `tcp4`, or `tcp6`.  It calls an external `LookupLoopbackIP()` function (not defined here) to retrieve available loopback IPs. If any listener creation fails, all created listeners are closed before returning an error.  
  
---  
  
### ListenPacketLoopback   
  
Similar to `ListenLoopback`, but creates and returns a slice of UDP packet connections instead of TCP listeners. It validates that the provided network type is one of `udp`, `udp4`, or `udp6`.  It also relies on the external `LookupLoopbackIP()` function for loopback IP addresses, and handles listener cleanup in case of errors during creation.  
  
# util/xnet/quic.go  
## xnet Package Component Summary  
  
**Package Name:** `xnet`  
  
**Imports:**  
*   `crypto/tls`: For TLS configuration related to QUIC connections.  
*   `net`: Provides basic networking primitives like addresses and listeners.  
*   `github.com/lucas-clemente/quic-go`: Core QUIC implementation library.  
*   `github.com/lucas-clemente/quic-go/qerr`:  QUIC error handling utilities.  
*   `github.com/sonm-io/core/util/multierror`: Utility for aggregating multiple errors into one.  
  
**External Data / Input Sources:**  
*   Network address (string) and network type (string) are used in `ListenQUIC`.  
*   TLS configuration (`tls.Config`) is required to secure QUIC connections via `ListenQUIC`.  
*   QUIC configuration (`quic.Config`) defines the behavior of the QUIC listener/connection, including supported versions and keep-alive settings.  
  
**TODOs:** None found in this file.  
  
---  
  
### Error Handling  
  
The code introduces a custom error type `quicError` that wraps standard errors to provide additional context (specifically timeout or temporary status). The `newQUICError` function creates instances of this wrapper, and methods like `Timeout()` and `Temporary()` attempt to extract QUIC-specific information from the underlying error using `qerr.ToQuicError`.  
  
### Default Configuration  
  
The `DefaultQUICConfig` function returns a preconfigured `quic.Config` object with specific supported QUIC versions (GQUIC39, GQUIC43, Milestone0_10_0) and enables keep-alive functionality. This provides a sensible default for creating QUIC connections without manual configuration.  
  
### Connection Management (`QUICConn`)  
  
The `QUICConn` struct wraps a `quic.Stream` and its associated `quic.Session`, providing an abstraction over the underlying QUIC connection. The `NewQUICConn` function creates new instances of this wrapper given a session, while methods like `LocalAddr()`, `RemoteAddr()`, and `Close()` delegate to the wrapped `session`.  The `Close()` method uses `multierror` to handle potential errors from closing both the stream and the session.  
  
### Listener Implementation (`QUICListener`)  
  
The `ListenQUIC` function creates a QUIC listener by first listening on a UDP socket using `net.ListenPacket`, then wrapping it with `quic.Listen`. The resulting listener is wrapped in a custom `QUICListener` struct for potential extension or customization.  The `Accept()` method within the `QUICListener` continuously accepts incoming sessions, and streams from those sessions. It handles peer-gone errors by skipping them (continuing to accept new connections) before returning an error if other issues occur. The `isPeerGoneErr` helper function checks for specific QUIC errors indicating a peer disconnection.  
  
# util/xnet/resolve.go  
Package: `xnet`  
  
Imports:  
- `bytes`  
- `fmt`  
- `io/ioutil`  
- `net`  
- `net/http`  
- `time`  
  
External Data / Input Sources:  
- HTTP endpoint (default: "http://checkip.amazonaws.com/") for resolving external public IP address.  The URL is configurable via the `NewExternalPublicIPResolver` function. The resolver fetches data from this source to determine the caller's public IP.  
  
TODOs:  
- None found in provided code snippet.  
  
## Code Summary  
  
### Loopback IP Lookup (`LookupLoopbackIP`)  
This function retrieves loopback IPv4 and IPv6 addresses by iterating through network interfaces. It filters for up and loopback interfaces, parses their addresses, and appends valid loopback IPs to separate slices (IPv6 first, then IPv4). The function returns a combined slice of these addresses or an error if interface retrieval fails.  
  
### External Public IP Resolver (`ExternalPublicIPResolver`)  
This struct provides a cached mechanism for resolving the caller's public IP address via HTTP requests to an external service. It caches the resolved IP and refreshes it after a configurable duration (default 10 minutes). The `NewExternalPublicIPResolver` function initializes the resolver with an optional URL, defaulting to "http://checkip.amazonaws.com/".  The `PublicIP()` method returns the cached or refreshed IP address. The resolver uses HTTP GET requests and parses the response body as a string representing the IP address.  
  
