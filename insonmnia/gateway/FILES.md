# insonmnia/gateway/gateway.go  
## Package: `gateway`  
  
**Imports:**  
  
*   `errors`: For custom error definitions.  
*   `net`: For network-related operations (IP address resolution).  
*   `strings`: For string manipulation (lowercasing protocol names).  
*   `syscall`: For accessing system call constants (protocol numbers like `IPPROTO_TCP`, `IPPROTO_UDP`).  
  
**External Data/Input Sources:**  
  
*   Hostnames: Used for DNS lookups to resolve IP addresses. Input via strings.  
*   Port Numbers:  Unsigned 16-bit integers, used for service and real endpoint definitions.  
*   Protocols: Strings ("tcp", "udp") that are converted into protocol numbers using `syscall`.  
*   Weights: Integer values representing the weight of a real endpoint in scheduling algorithms.  
  
**TODOs:**  
  
None found in this code snippet.  
  
---  
  
### Service Options (`ServiceOptions` struct and `NewServiceOptions` function)  
  
This section defines the structure for virtual service options, including host, port, protocol, method (defaulting to "wrr"), and persistence flag. The `NewServiceOptions` function constructs these options, performing DNS resolution on the provided hostname if present. It handles validation errors such as missing endpoints or unknown protocols.  The resolved IP address is stored in the `host` field of the struct. Protocol strings are converted to their corresponding syscall protocol numbers (TCP/UDP).  
  
### Real Options (`RealOptions` struct and `NewRealOptions` function)  
  
This section defines the structure for real service options, including host, port, weight, and virtual service ID. The `NewRealOptions` function constructs these options, performing DNS resolution on the provided hostname if present. It handles validation errors such as missing endpoints.  The resolved IP address is stored in the `host` field of the struct. Weight defaults to 100 if a non-positive value is given.  
  
# insonmnia/gateway/gateway_linux.go  
## Gateway Component Summary  
  
**Package Name:** `gateway`  
  
**Imports:**  
  
*   `context`: For managing request contexts.  
*   `errors`: For defining custom error types.  
*   `fmt`: For formatted string output.  
*   `net`: For network operations, including IP address handling.  
*   `sync`: For synchronization primitives (mutexes).  
*   `github.com/tehnerd/gnl2go`:  IPVS client library for interacting with the Linux kernel's IP Virtual Server.  
*   `github.com/noxiouz/zapctx/ctxlog`: Structured logging using Zap with context awareness.  
*   `go.uber.org/zap`: For structured logging.  
  
**External Data / Input Sources:**  
  
*   Service Options (`ServiceOptions` struct): Host, Port, Protocol, Method for virtual services.  
*   Real Options (`RealOptions` struct): Host, Port, Weight, methodID for backends.  
*   IPVS Configuration: The component directly interacts with the Linux kernel's IP Virtual Server (IPVS) using `gnl2go`.  
  
**TODO Comments:**  
  
*   Implement `GetServices()` to return a list of virtual service IDs.  
*   Implement `GetBackends(vsID string)` to retrieve backends associated with a specific virtual service ID.  
*   Implement `GetBackend(vsID, rsID string)` to fetch the RealOptions for a given backend within a virtual service.  
  
**Code Summary:**  
  
### Gateway Initialization (`NewGateway`)  
  
The `NewGateway` function initializes the gateway by creating an IPVS client instance using `gnl2go`. It flushes existing IPVS rules (potentially dangerous if not handled carefully) and sets up internal maps for tracking services and backends.  Errors during initialization result in cleanup via a `Close()` call before returning.  
  
### Service Management (`CreateService`, `RemoveService`)  
  
The component allows creating, removing virtual services using the underlying IPVS infrastructure. The `createService` function adds a new service to IPVS based on provided options and stores it internally.  `removeService` deregisters a service from IPVS, cleans up orphaned backends associated with that service, and removes internal tracking data.  
  
### Backend Management (`CreateBackend`, `RemoveBackend`)  
  
Backends are registered and unregistered using the `createBackend` and `removeBackend` functions. These operations interact directly with IPVS to add or remove destination ports (backends) associated with a virtual service.  Error handling includes checks for existing backends and missing services.  
  
### Metrics Collection (`GetMetrics`)  
  
The `GetMetrics` function retrieves statistics from IPVS for a given virtual service, including connection counts, packet rates, and byte transfer rates. It relies on the `ipvs.GetAllStatsBrief()` method to gather data.  If no stats are found for the specified service, it returns an error.  
  
### Shutdown (`Close`)  
  
The `Close` function gracefully shuts down the gateway by removing all registered services (and their associated backends) from IPVS and exiting the IPVS client. This ensures that resources are released properly.  
  
### Utility Function (`GetOutboundIP`)  
This function attempts to determine the outbound IP address of the system by establishing a UDP connection to an external server (8.8.8.8). It's used for obtaining the local IP address from the established connection.  
  
# insonmnia/gateway/gateway_nonlinux.go  
## Package: `gateway`  
  
**Imports:**  
  
*   `context`  
  
**External Data/Input Sources:** None. The code does not interact with external data sources or accept any input beyond the context passed to `NewGateway`.  
  
**TODOs:** None found in this file.  
  
---  
  
### Core Functionality Summary:  
  
This file defines a basic gateway component, likely intended for network traffic management (though minimal implementation is present). It provides a simple struct (`Gateway`) and associated functions for creation (`NewGateway`) and cleanup (`Close`). The `PlatformSupportIPVS` constant suggests potential future integration with IP Virtual Server (IPVS) load balancing on Linux systems, but it's currently disabled.  
  
The `NewGateway` function simply returns an initialized instance of the `Gateway` struct without any error handling or configuration.  The `Close` method is a no-op; it does nothing when called. The build tag `// +build !linux` indicates that this code path will not be compiled on Linux systems, suggesting platform-specific implementations may exist elsewhere in the package.  
  
---  
  
# insonmnia/gateway/metrics.go  
## Package/Component Name and Imports  
  
**Package Name:** `gateway`  
  
**Imports:** None  
  
## External Data / Input Sources  
  
This code does not directly handle external data or input sources. It operates on in-memory `Metrics` structs, likely populated by other parts of the system (not shown here). The metrics themselves represent aggregated network statistics.  
  
## TODOs  
  
There are no `TODO` comments in this file.  
  
---  
  
### Data Structures: `Metrics`  
  
The code defines a struct named `Metrics`. This structure holds cumulative and per-second counts for network connections, packets (inbound/outbound), and bytes transferred (inbound/outbound). The purpose is to track virtual service performance metrics.  It's designed for aggregation; the `Add` method allows combining multiple metric instances into a single one.  
  
### Function: `Add`  
  
The `Add` function is a method on the `Metrics` struct that performs in-place addition of another `Metrics` instance. It accumulates all tracked fields (connections, packets, bytes) and their per-second rates. This suggests this component is part of an aggregation pipeline where metrics from different sources are combined over time.  
  
# insonmnia/gateway/pool.go  
## Package/Component Name: `gateway`  
  
**Imports:**  
  
*   `errors`: For error handling.  
*   `math/rand`: For generating random numbers (used for initial port assignment).  
*   `sync`: For mutex locking to ensure thread-safe access to the port pool.  
*   `gopkg.in/oleiade/lane.v1`:  A queue implementation used for managing available ports.  
  
**External Data / Input Sources:**  
  
*   `init uint16`: Initial value of the first port in the range.  
*   `size uint16`: The total number of ports to manage within the pool.  
*   `ID string`: Identifier passed during `Assign` and `Retain`.  
  
**TODOs:** None found in this file.  
  
---  
  
### Summary: PortPool Structure & Initialization  
  
The code defines a `PortPool` struct that manages a fixed-size set of ports using a queue (`lane.Queue`) to track available ports and a map (`used`) to keep track of assigned ports by ID. The `NewPortPool` function initializes the pool with a specified initial value (`init`) and size (`size`). It populates the queue with random port numbers within the defined range, ensuring that each port is unique at initialization.  
  
### Summary: Port Assignment (`Assign`)  
  
The `Assign` method attempts to assign an available port from the queue to a given ID.  It uses a mutex lock to prevent race conditions when accessing shared data (queue and used map). If the ID already exists in the `used` map, it returns an error. Otherwise, it dequeues a port from the queue, assigns it to the ID in the `used` map, and returns the assigned port number.  If the queue is empty (no ports left), it returns an error.  
  
### Summary: Port Retention (`Retain`)  
  
The `Retain` method releases a previously assigned port back into the pool. It uses a mutex lock for thread safety. If the ID exists in the `used` map, the corresponding port number is retrieved, enqueued back into the queue (making it available again), and removed from the `used` map.  If the ID does not exist, it returns an error indicating that the port was never assigned.  
  
# insonmnia/gateway/pool_test.go  
## Package/Component Name: `gateway`  
  
**Imports:**  
  
*   `testing`: Standard testing package for Go.  
*   `github.com/stretchr/testify/assert`: Assertion library used in tests.  
  
**External Data / Input Sources:** None explicitly defined within this file; the tests rely on internal state and function calls to `NewPortPool`.  
  
**TODOs:** None found.  
  
---  
  
### Code Summary:  
  
This file contains unit tests for a `PortPool` implementation (presumably defined elsewhere in the package). The tests cover various scenarios related to port assignment, retention, and error handling within the pool.   
  
*   **TestPool**: Verifies that assigning ports works correctly when available, and returns an error when no ports are left. It checks if assigned ports fall within the expected range (10 or 11 in this case).  
*   **TestPoolRetainWhileEmpty**: Tests the behavior of retaining a port when the pool is empty; it expects an error to be returned.  
*   **TestPoolDoubleAssign**: Checks that attempting to assign the same port twice results in an error, as expected for a resource pool.  
*   **TestPoolAssignRetainAssign**: Validates that assigning a port, retaining it, and then re-assigning it works correctly (i.e., returns the same port again).  
  
The tests use `assert` from the `testify` library to verify expected outcomes, including error conditions and assigned port values. The core functionality being tested revolves around managing a limited set of ports and ensuring proper allocation/deallocation behavior.  
  
# insonmnia/gateway/router.go  
## Package/Component Summary: `gateway`  
  
**Package Name:** `gateway`  
  
**Imports:** None.  
  
**External Data/Input Sources:** The code relies on external input for virtual service and real endpoint configuration (IDs, hosts, ports). These are likely provided through some higher-level orchestration or management system not present in this file itself.  The protocol string is also an external parameter.  
  
**TODOs:**  
*   "I think this is the first candidate for package decomposition. Not going to do this right now to avoid unreviewable diff." - Suggests potential refactoring into smaller, more manageable components.  
  
---  
  
### Core Interfaces: `VirtualService` and `Router`  
  
The `VirtualService` interface defines methods for managing real service endpoints (adding/removing) associated with a virtual service ID. The `Router` interface provides higher-level operations like registering/deregistering virtual services, retrieving metrics, and closing the router. These interfaces are central to how gateway functionality is exposed.  
  
### Direct Router Implementation: `directRouter` & `directVirtualService`  
  
The code implements a basic "direct" router (`directRouter`) that immediately returns a new `directVirtualService` upon registration. The `directVirtualService` simply stores the ID and protocol, and its `AddReal` method creates a `Route` struct with provided parameters without any validation or complex logic.  All other methods in both implementations are stubs returning nil errors. This suggests it's either a placeholder or an extremely simplified implementation for testing/development purposes.  
  
### Route Struct  
  
The `Route` struct holds the configuration details of backend service endpoints, including ID, protocol, host, port, and backend-specific information. It serves as the data structure used to represent routing destinations within the gateway.  
  
---  
  
# insonmnia/gateway/router_linux.go  
```markdown  
## Package: `gateway` - IPVS Router Implementation Summary  
  
**Package Name:** `gateway`  
  
**Imports:**  
*   `context`: For context management in asynchronous operations.  
*   `sync`: For thread-safe access to shared data structures (mutex).  
  
**External Data/Input Sources:**  
*   `Gateway`: Dependency on the main gateway component for service creation, removal and metrics retrieval.  
*   `PortPool`: Dependency for assigning ports to services.  
*   `GetOutboundIP()`: External function call to retrieve outbound IP address.  This is a critical external dependency that could affect functionality if unavailable or incorrect.  
  
**TODOs:** None found in the provided code snippet.  
  
---  
  
### Core Functionality: `ipvsRouter` Structure and Lifecycle  
  
The `ipvsRouter` struct implements an IPVS-based router within the gateway system. It manages virtual services, their associated metrics, and interacts with a `Gateway` instance for service lifecycle operations (creation, removal). The structure uses a mutex (`mu`) to ensure thread safety when accessing shared data like `metrics` and `services`.  
  
The `newIPVSRouter` function initializes an `ipvsRouter` instance.  The router's primary functions include registering new virtual services (`Register`), deregistering existing ones (`Deregister`, `deregisterRoute`), retrieving metrics (`GetMetrics`, `getMetrics`, `updateServiceMetrics`), and closing all routes (`Close`).  
  
---  
  
### Service Registration & Deregistration: `Register` and `Deregister` Methods  
  
The `Register` method creates a new virtual service by assigning a port from the `PortPool`, creating the corresponding service in the underlying `Gateway`, and storing it internally.  Error handling is present for failures during IP retrieval, port assignment, or gateway interaction. The `Deregister` method removes an existing route via `deregisterRoute`.  
  
---  
  
### Metrics Collection: `GetMetrics` & Related Functions  
  
The router collects network-specific metrics associated with each virtual service using the `GetMetrics`, `getMetrics`, and `updateServiceMetrics` methods.  It iterates through registered services, retrieves their metrics from the `Gateway`, and aggregates them into a combined `Metrics` object. The mutex ensures thread safety during metric retrieval and aggregation.  
  
---  
  
### Virtual Service Interaction: `ipvsVirtualService` Structure & Methods  
  
The `ipvsVirtualService` struct represents an individual virtual service managed by the router. It stores its ID, associated options (`ServiceOptions`), and a reference to the parent `Gateway`. The methods `AddReal` and `RemoveReal` allow adding or removing backend servers (real instances) for the virtual service via interactions with the underlying `Gateway`.  The `ID()` method returns the service's identifier.  
  
---  
  
### Backend Management: `AddReal`, `RemoveReal` Methods  
  
These functions manage backends associated with a virtual service by creating/removing them in the gateway using `CreateBackend` and `RemoveBackend` calls respectively. The backend host, port, and protocol are configured via `NewRealOptions`.  
```  
<end_of_output>  
  
# insonmnia/gateway/router_nonlinux.go  
## Package/Component Name: `gateway`  
  
**Imports:**  
  
*   `context`  
  
**External Data / Input Sources:**  
  
*   The function `newIPVSRouter` takes a `context.Context`, a pointer to a `Gateway` struct, and a pointer to a `PortPool` struct as input. The exact structure of these types is not defined within this snippet but are assumed to be part of the larger package or external dependencies.  
  
**TODOs:**  
  
*   None found in this code snippet.  
  
---  
  
### Code Summary:  
  
The provided code defines a function `newIPVSRouter` that returns a `Router` instance. The implementation is conditional, built only when not compiling for Linux (`// +build !linux`).  It appears to be a placeholder or fallback mechanism; it simply calls `newDirectRouter()` regardless of the input parameters (context, Gateway, PortPool). This suggests that on non-Linux systems, IPVS routing is either unavailable or bypassed in favor of direct routing. The function's purpose seems to provide an alternative router implementation when IPVS isn't applicable.  
  
