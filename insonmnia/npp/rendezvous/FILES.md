# insonmnia/npp/rendezvous/client.go  
## Rendezvous Package Summary  
  
**Package Name:** `rendezvous`  
  
**Imports:**  
  
*   `context`: For managing request contexts.  
*   `github.com/sonm-io/core/proto`: Contains generated gRPC definitions for the Sonm core protocol.  
*   `github.com/sonm-io/core/util/xgrpc`: Utility functions for creating and managing gRPC clients.  
*   `google.golang.org/grpc`: Core gRPC library.  
*   `google.golang.org/grpc/credentials`: Credentials management for secure gRPC connections.  
  
**External Data / Input Sources:**  
  
*   `addr` (string): The address of the gRPC server to connect to.  This is a required input when creating a new client.  
*   `credentials` (`credentials.TransportCredentials`): Optional credentials used for authentication with the gRPC server.  
*   `opts ...grpc.DialOption`: Additional dial options that can be passed to `xgrpc.NewClient`.  
  
**TODOs:** None found in this file.  
  
### Client Interface  
  
The `Client` interface extends the generated `sonm.RendezvousClient` and adds a `Close()` method for explicitly closing the underlying gRPC connection. This allows users to terminate pending operations cleanly.  
  
### Client Implementation (`client`)  
  
The `client` struct wraps the generated `sonm.RendezvousClient` and stores a reference to the underlying `grpc.ClientConn`.  This is used by the `Close()` method to shut down the connection.  
  
### NewRendezvousClient Function  
  
This function creates a new rendezvous client using `xgrpc.NewClient` to establish a gRPC connection. It returns an instance of the custom `client` struct, which implements the `Client` interface.  Error handling is included for connection failures.  
  
### Close Method  
  
The `Close()` method on the `client` struct simply calls `m.conn.Close()`, terminating the underlying gRPC connection and any pending operations.  
  
# insonmnia/npp/rendezvous/config.go  
## Rendezvous Package Summary  
  
**Package Name:** `rendezvous`  
  
**Imports:**  
*   `crypto/ecdsa`: For elliptic curve cryptography, specifically private key handling.  
*   `net`:  For network address manipulation.  
*   `time`: For duration-based configurations (timeouts).  
*   `github.com/jinzhu/configor`: For loading configuration from files (YAML support).  
*   `github.com/sonm-io/core/accounts`: For Ethereum account management and key loading.  
*   `github.com/sonm-io/core/insonmnia/auth`:  For authentication related address configurations.  
*   `github.com/sonm-io/core/insonmnia/logging`: For logging configuration.  
*   `github.com/sonm-io/core/util/debug`: For debugging settings.  
*   `github.com/sonm-io/core/util/netutil`:  For network utility functions, specifically TCP address handling.  
  
**External Data / Inputs:**  
*   Configuration file path (string) for `NewServerConfig`.  
*   YAML configuration files defining server and client settings. These include endpoint addresses, Ethereum key paths, logging configurations, debugging options, connection attempt limits, and timeouts.  
  
**TODOs:** None found in the provided code snippet.  
  
### Server Configuration (`ServerConfig`, `serverConfig`)  
  
The package defines structures for configuring a Rendezvous server.  `ServerConfig` is the public interface, while `serverConfig` is used internally by `configor`. The configuration includes:  
*   Listening address (`Addr`).  
*   Ethereum private key (`PrivateKey`).  
*   Logging settings (`Logging`).  
*   Debugging options (`Debug`).  
  
The `NewServerConfig` function loads these settings from a YAML file, validates the Ethereum key, and returns a populated `ServerConfig`.  It uses `configor.Load` to parse the configuration file into the internal `serverConfig` struct before converting it to the public `ServerConfig`.  
  
### Global Configuration (`Config`)  
The package also defines a global `Config` structure for client-side settings:  
*   A list of endpoints (`Endpoints`) used for connection attempts (type `auth.Addr`).  
*   Maximum number of connection attempts (`MaxConnectionAttempts`, default 5).  
*   Timeout duration for connections (`Timeout`, default 3 seconds).  
  
These global configurations are loaded from YAML files using the same `configor` library, allowing centralized control over client behavior.  
  
# insonmnia/npp/rendezvous/options.go  
## Rendezvous Package Summary  
  
**Package Name:** `rendezvous`  
  
**Imports:**  
  
*   `crypto/tls`: For TLS configuration related to secure connections.  
*   `github.com/sonm-io/core/util/xgrpc`: Custom gRPC transport credentials implementation.  
*   `go.uber.org/zap`: Structured logging library.  
  
**External Data / Input Sources:**  
  
*   `tls.Config`: Used for configuring TLS credentials, enabling secure connections.  Nil values are supported but discouraged as they disable authentication.  
*   `zap.Logger`: Logger instance used for internal logging within the package. Can be nil to deactivate logging entirely.  
  
**Options Configuration:**  
  
The `rendezvous` package uses functional options (`Option` type) to configure server behavior:  
  
*   `WithLogger(log *zap.Logger)`: Sets the logger instance.  
*   `WithCredentials(cfg *tls.Config)`: Configures TLS credentials for secure connections (required when using QUIC).  
*   `WithQUIC()`: Enables QUIC support, requiring transport credentials to be set via `WithCredentials`.  
  
**Major Code Parts Summary:**  
  
1.  **Options Struct (`options`)**: Holds configuration parameters for the rendezvous server including logger, credentials and quic enable flag. Default values are provided in `newOptions()`.  
2.  **Option Function Type**: Defines a function type that takes an options pointer as input allowing functional style configuration of the server.  
3.  **Configuration Options Functions (`WithLogger`, `WithCredentials`, `WithQUIC`)**: These functions return Option instances to configure the server's behavior, providing flexibility in setting up logging, credentials and quic support.  
  
**TODOs:**  
  
No TODO comments found in this file.  
  
# insonmnia/npp/rendezvous/peer.go  
## rendezvous Package Summary  
  
This package, `rendezvous`, provides functionality for managing peer connections and identifying them uniquely within a network context. It's designed to handle scenarios where standard Ethereum addresses are insufficient as unique identifiers (e.g., multiple servers sharing the same address). The core concept revolves around assigning each connected peer a randomly generated UUID-based ID (`PeerID`) for tracking purposes, especially during request lifecycle management.  
  
**Imports:**  
  
*   `github.com/pborman/uuid`: Used for generating universally unique identifiers (UUIDs) to create `PeerID` values.  
*   `github.com/sonm-io/core/proto`:  Likely used for defining address structures (`sonm.Addr`) associated with peers, though the exact usage isn't visible in this snippet.  
*   `google.golang.org/grpc/peer`: Provides access to gRPC peer information (e.g., remote address, authentication context).  
  
**Data Structures:**  
  
*   `Peer`: A struct that wraps a `grpc.Peer` instance along with a unique `PeerID` and a slice of private addresses (`[]*sonm.Addr`). This structure represents a connected peer in the network.  
*   `PeerID`:  A custom type (string alias) representing a unique identifier for peers, generated using UUIDs.  
  
**Functions:**  
  
*   `NewPeer(peerInfo peer.Peer, privateAddrs []*sonm.Addr) Peer`: Constructs a new `Peer` instance given gRPC peer information and private addresses.  
*   `NewPeerID() PeerID`: Generates a new unique `PeerID` using the `uuid` library.  This is the primary mechanism for assigning identifiers to peers.  
  
**External Data/Inputs:**  
  
*   `peer.Peer`: Input from gRPC framework, representing peer connection details.  
*   `[]*sonm.Addr`: Private network addresses associated with a peer (likely used for direct communication).  
  
**TODOs:**  
  
None found in this code snippet.  
  
**Summary of Major Code Parts:**  
  
The package focuses on creating and managing unique identifiers (`PeerID`) for peers connected via gRPC. The `Peer` struct combines standard gRPC peer information with the generated ID and private addresses, providing a complete representation of a network participant.  The use of UUIDs ensures uniqueness even in scenarios where other identifiers (like Ethereum addresses) might be shared across multiple instances. This is likely part of a larger system for managing distributed resources or coordinating tasks between peers.  
  
# insonmnia/npp/rendezvous/server.go  
## Rendezvous Protocol Implementation Summary  
  
This document summarizes the `rendezvous` package, which implements a bidirectional locator protocol for mutual address resolution between peers, particularly useful in NAT environments. The core functionality revolves around facilitating P2P connections by exchanging public and private network addresses.  The package provides server-side components for managing meetings (sessions) between clients and servers.  
  
### Package Information  
  
*   **Package Name:** `rendezvous`  
*   **Imports:**  
    *   `context`: For handling request contexts.  
    *   `errors`: For error management.  
    *   `fmt`: For formatted output.  
    *   `math/rand`: For random peer selection.  
    *   `net`: For network operations (TCP, QUIC).  
    *   `sync`: For concurrent access control (mutexes).  
    *   `time`: For time-based logic (timeouts, expiration).  
    *   `github.com/ethereum/go-ethereum/common`: Ethereum address utilities.  
    *   `github.com/ethereum/go-ethereum/crypto`: Cryptographic functions.  
    *   `github.com/sonm-io/core/insonmnia/auth`: Authentication related functionality.  
    *   `github.com/sonm-io/core/insonmnia/logging`: Logging utilities.  
    *   `github.com/sonm-io/core/insonmnia/npp/nppc`: NPP (Network Peer Protocol) resource ID definitions.  
    *   `github.com/sonm-io/core/proto`: Protocol buffer definitions.  
    *   `github.com/sonm-io/core/util/debug`: Debugging utilities.  
    *   `github.com/sonm-io/core/util/xgrpc`: Extended gRPC functionalities.  
    *   `github.com/sonm-io/core/util/xnet`: Network utility functions.  
    *   `go.uber.org/zap`: Structured logging library.  
    *   `golang.org/x/sync/errgroup`: Error handling for concurrent operations.  
    *   `google.golang.org/grpc`: gRPC framework.  
    *   `google.golang.org/grpc/codes`: gRPC error codes.  
    *   `google.golang.org/grpc/keepalive`: gRPC keep-alive parameters.  
    *   `google.golang.org/grpc/peer`: Accessing peer information in gRPC contexts.  
  
### External Data & Inputs  
  
*   **Server Configuration (`ServerConfig`)**: Defines server address, private key for authentication, and debug settings.  
*   **gRPC Requests:** `ConnectRequest`, `ID`. Used to initiate rendezvous requests.  
*   **Peer Information:** Obtained from gRPC context via `peer.FromContext` or `auth.FromContext`. Includes peer ID, public/private addresses.  
  
### TODOs  
  
*   Track IP version during resolution (IPv4 vs IPv6) to avoid returning incompatible addresses.  
  
### Major Code Parts Summary  
  
#### 1. Meeting Management (`meeting` struct)  
  
The `meeting` structure manages sessions between clients and servers for a specific NPP resource ID. It uses mutexes to ensure thread-safe access to client/server lists. Methods include:  
  
*   `AddServer`, `RemoveServer`: Registering/unregistering servers in the meeting.  
*   `AddClient`, `RemoveClient`: Managing waiting clients.  
*   `PopRandomServer`, `PopRandomClient`: Selecting random peers for connection attempts.  
*   `IsServerInactive`: Checks if all servers have been inactive for a defined duration.  
  
#### 2. Server Implementation (`Server` struct)  
  
The `Server` structure represents the rendezvous server itself. It handles gRPC requests, manages meetings, and facilitates address exchange between peers. Key methods include:  
  
*   `NewServer`: Constructs a new server instance with configurable options (TLS credentials, logging).  
*   `Resolve`, `Publish`: Core RPC handlers for resolving/publishing peer addresses.  
*   `Run`: Starts the gRPC server over TCP and optionally QUIC.  
*   `Stop`: Shuts down the server gracefully.  
  
#### 3. Address Resolution Logic  
  
The `Resolve` and `Publish` methods handle address exchange:  
  
1.  Clients call `Resolve` to find a matching server for their ID.  
2.  Servers register themselves using `Publish`.  
3.  If a match is found, addresses are exchanged directly.  
4.  If no direct connection is possible (NAT), TCP punching or relay servers may be used (not fully implemented in this code).  
  
#### 4. gRPC Integration  
  
The server uses gRPC for communication:  
  
*   `xgrpc.NewServer`: Creates a gRPC server with custom interceptors (logging, tracing, authentication).  
*   Keep-alive parameters are configured to maintain active connections.  
*   TLS credentials can be provided for secure communication.  
  
#### 5. Debugging and Monitoring  
  
The `debug` package is used for exposing PProf endpoints for performance monitoring. Logging is handled using the `zap` library.  
  
