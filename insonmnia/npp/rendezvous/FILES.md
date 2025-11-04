# insonmnia/npp/rendezvous/client.go  
## Package: `rendezvous`  
  
**Imports:**  
  
*   `context`  
*   `github.com/sonm-io/core/proto` (sonm generated protobuf definitions)  
*   `github.com/sonm-io/core/util/xgrpc` (utility for gRPC client creation)  
*   `google.golang.org/grpc` (core gRPC library)  
*   `google.golang.org/grpc/credentials` (gRPC credentials handling)  
  
**External Data/Input Sources:**  
  
*   `addr` (string): The address of the gRPC server.  
*   `credentials` (credentials.TransportCredentials): Credentials for authentication.  
*   `opts ...grpc.DialOption`: Additional gRPC dial options.  
*   `ctx` (context.Context): Context for gRPC operations.  
  
**TODOs:**  
  
*   None found in this file.  
  
---  
  
### Summary  
  
This package provides a wrapper around the generated gRPC `RendezvousClient` from the `sonm.proto` definitions. The primary purpose is to allow explicit closing of the underlying gRPC connection, which is not directly exposed by the generated client.  
  
The `Client` interface extends the generated `sonm.RendezvousClient` with a `Close()` method. The `NewRendezvousClient` function creates a new client instance, establishing a gRPC connection using `xgrpc.NewClient` and wrapping the generated client. The `client` struct holds both the generated client and the underlying `grpc.ClientConn` to enable the `Close()` method to terminate the connection. The `Close()` method simply calls `conn.Close()`, ensuring resource cleanup.  
  
The package is designed to provide more control over the gRPC connection lifecycle, allowing users to explicitly terminate pending operations when needed.  
  
# insonmnia/npp/rendezvous/config.go  
## Rendezvous Package Component Summary  
  
**Package Name:** `rendezvous`  
  
**Imports:**  
- `crypto/ecdsa`: For elliptic curve digital signature algorithm operations (private key handling).  
- `net`: For network address handling.  
- `time`: For duration-related configurations (timeouts).  
- `github.com/jinzhu/configor`: For loading configuration from files (YAML).  
- `github.com/sonm-io/core/accounts`: For Ethereum account management (key loading).  
- `github.com/sonm-io/core/insonmnia/auth`: For authentication-related address configurations.  
- `github.com/sonm-io/core/insonmnia/logging`: For logging configuration.  
- `github.com/sonm-io/core/util/debug`: For debugging configuration.  
- `github.com/sonm-io/core/util/netutil`: For network utility functions (TCP address handling).  
  
**External Data/Input Sources:**  
- **Configuration File (YAML):** The primary input source is a YAML configuration file loaded using `configor`. This file defines server settings, Ethereum account details, logging, and debugging options.  
- **Ethereum Private Key:** The configuration relies on an Ethereum private key loaded from the `EthConfig` section of the YAML file.  
- **Network Address:** The server's listening address is specified in the configuration file.  
  
**TODOs:**  
- None found in the provided code snippet.  
  
**Code Part Summaries:**  
  
### Server Configuration (`ServerConfig`, `serverConfig`, `NewServerConfig`)  
This section defines the structure for the Rendezvous server configuration. `ServerConfig` is the main struct, while `serverConfig` is used for YAML loading. `NewServerConfig` loads the configuration from a YAML file, validates it, loads the Ethereum private key, and returns a populated `ServerConfig` instance. The configuration includes the listening address, Ethereum private key, logging settings, and debugging settings.  
  
### General Configuration (`Config`)  
This section defines a general configuration struct (`Config`) used for client-side or other components interacting with the Rendezvous server. It includes a list of endpoints (authentication addresses), maximum connection attempts, and a timeout duration. These settings control how clients connect to the Rendezvous service.  
  
# insonmnia/npp/rendezvous/options.go  
## Package: `rendezvous`  
  
**Imports:**  
  
*   `crypto/tls`: For TLS configuration.  
*   `github.com/sonm-io/core/util/xgrpc`: For gRPC transport credentials.  
*   `go.uber.org/zap`: For structured logging.  
  
**External Data/Input Sources:**  
  
*   `*zap.Logger`: Logger instance for internal logging. Can be nil to disable logging.  
*   `*tls.Config`: TLS configuration for secure connections.  Nil disables authentication (discouraged).  
*   `bool`: Enables or disables QUIC support.  
  
**TODOs:**  
  
*   None found in this file.  
  
---  
  
### Options Configuration  
  
The code defines an `options` struct to hold configuration parameters for the rendezvous server. These parameters include a logger (`*zap.Logger`), transport credentials (`*xgrpc.TransportCredentials`), and a flag to enable QUIC (`bool`). The `newOptions` function initializes the struct with default values (no logger, no credentials).  
  
### Functional Options Pattern  
  
The code implements the functional options pattern using the `Option` type (a function that modifies the `options` struct). This allows for flexible configuration of the server through a series of option functions:  
  
*   `WithLogger`: Sets the logger.  
*   `WithCredentials`: Sets the transport credentials (TLS configuration).  
*   `WithQUIC`: Enables QUIC support.  Requires credentials to be set.  
  
The documentation emphasizes the importance of providing credentials when enabling QUIC, as QUIC relies on TLS-equivalent security.  
  
# insonmnia/npp/rendezvous/peer.go  
## Package: `rendezvous`  
  
**Imports:**  
  
*   `github.com/pborman/uuid`: Used for generating unique peer IDs.  
*   `github.com/sonm-io/core/proto`: Used for `sonm.Addr` type.  
*   `google.golang.org/grpc/peer`: Used for accessing peer information.  
  
**External Data/Input Sources:**  
  
*   `peer.Peer`: Input from gRPC peer connection.  
*   `[]*sonm.Addr`: Input list of private addresses.  
  
**TODOs:**  
  
*   None found in this file.  
  
**Summary of Code Parts:**  
  
### `Peer` Struct  
  
The `Peer` struct wraps `grpc.Peer` and adds a unique `PeerID` and a slice of private addresses (`[]*sonm.Addr`). This structure is used to identify peers within the rendezvous system, providing a unique identifier beyond the Ethereum address.  
  
### `NewPeer` Function  
  
The `NewPeer` function constructs a new `Peer` instance, taking a `grpc.Peer` and a list of private addresses as input. It generates a new `PeerID` using `NewPeerID()` and initializes the `Peer` struct.  
  
### `PeerID` Type and `NewPeerID` Function  
  
The `PeerID` type is a string alias used to represent a unique identifier for peers. The `NewPeerID` function generates a new UUID using the `uuid` package and converts it to a `PeerID` string. This ensures uniqueness even if peers share Ethereum addresses.  
  
### `PeerID.String()` Method  
  
The `String()` method on the `PeerID` type provides a string representation of the ID, allowing it to be easily printed or used in string-based operations.  
  
# insonmnia/npp/rendezvous/server.go  
## Rendezvous Package Summary  
  
**Package Name:** `rendezvous`  
  
**Imports:**  
  
*   `context`  
*   `errors`  
*   `fmt`  
*   `math/rand`  
*   `net`  
*   `sync`  
*   `time`  
*   `github.com/ethereum/go-ethereum/common`  
*   `github.com/ethereum/go-ethereum/crypto`  
*   `github.com/sonm-io/core/insonmnia/auth`  
*   `github.com/sonm-io/core/insonmnia/logging`  
*   `github.com/sonm-io/core/insonmnia/npp/nppc`  
*   `github.com/sonm-io/core/proto`  
*   `github.com/sonm-io/core/util/debug`  
*   `github.com/sonm-io/core/util/xgrpc`  
*   `github.com/sonm-io/core/util/xnet`  
*   `go.uber.org/zap`  
*   `golang.org/x/sync/errgroup`  
*   `google.golang.org/grpc`  
*   `google.golang.org/grpc/codes`  
*   `google.golang.org/grpc/keepalive`  
*   `google.golang.org/grpc/peer`  
*   `google.golang.org/grpc/status`  
  
**External Data/Input Sources:**  
  
*   **gRPC Connections:** The server accepts gRPC connections over TCP and optionally QUIC.  
*   **Peer Information:**  Relies on peer information extracted from gRPC context (using `peer.FromContext` and `auth.FromContext`).  
*   **External Public IP:** Uses `xnet.ExternalPublicIPResolver` to determine the public IP address.  
*   **Configuration:**  Takes a `ServerConfig` struct for address, TLS credentials, and debug settings.  
  
**TODOs:**  
  
*   `TODO: When resolving it's necessary to track also IP version. For example to be able not to return IPv6 when connecting socket is IPv4.`  
  
**Code Summary:**  
  
### Rendezvous Protocol Implementation  
  
The core of this package implements a bidirectional locator (rendezvous) protocol for mutual address resolution between peers, especially useful when direct connectivity is uncertain (e.g., behind NATs). The protocol allows servers to publish private network addresses while resolving their public addresses. Clients provide their private addresses, and the server facilitates the exchange of public and private addresses for both peers.  
  
### Data Structures  
  
*   `meeting`: Manages clients and servers for a specific NPP identifier. It stores peer candidates and tracks server activity.  
*   `peerCandidate`: Holds a `Peer` object and a channel for communication.  
*   `Server`: Represents the rendezvous server, handling connections, resolving addresses, and managing meetings.  
  
### Server Functionality  
  
*   `NewServer`: Constructs a new server with configurable options (TLS, logging).  
*   `Resolve`: Handles client requests to resolve a peer's address. It adds the client to a meeting and waits for a server match.  
*   `Publish`: Handles server requests to publish their address. It adds the server to a meeting and waits for a client match.  
*   `ResolveAll`: Returns all registered peers for a given ID.  
*   `Run`: Starts the server, listening for gRPC connections over TCP and optionally QUIC.  
*   `Stop`: Stops the server, closing connections and listeners.  
  
### Meeting Management  
  
The `meeting` struct manages clients and servers waiting to connect. It includes logic for popping random candidates, removing inactive servers, and cleaning up empty meetings.  
  
### Address Resolution Logic  
  
The server resolves addresses by checking if peers are in the same LAN (using private addresses) or attempting TCP punching if both are behind NATs. If all else fails, a relay server can be used.  
  
### Debugging and Logging  
  
The server includes debugging features (PProf) and uses `zap` for structured logging.  
  
