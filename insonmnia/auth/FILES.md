# insonmnia/auth/addr.go  
**Package/Component Name:** `auth`  
  
**Imports:**  
- `fmt` (for formatted I/O)  
- `strings` (for string manipulation)  
- `github.com/ethereum/go-ethereum/common` (for Ethereum address handling)  
  
**External Data/Input Sources:**  
- String input for parsing addresses (`ParseAddr`, `UnmarshalYAML`). The format is expected to be either a valid Ethereum hex address, a network address (e.g., "localhost:8080"), or both separated by "@".  
- YAML data when unmarshaling an `auth.Addr` object via `UnmarshalYAML`.  
  
**TODOs:** None found in the provided code snippet.  
  
---  
  
### Code Summary  
  
#### Address Structure (`Addr`)  
The `Addr` struct represents a unified address that can contain either an Ethereum common address, a network address, or both combined (separated by "@"). The structure encapsulates these addresses without making assumptions about their relationship; it's up to the user to verify if the network address corresponds to the provided Ethereum address.  
  
#### Address Parsing (`ParseAddr`)  
The `ParseAddr` function parses an input string into an `Addr` struct. It handles three cases: a single Ethereum address, a single network address, or both separated by "@". If it detects an invalid Ethereum address format, it returns an error. The parsing logic relies on the `common.IsHexAddress` and `common.HexToAddress` functions from the imported package to validate and convert Ethereum addresses.  
  
#### Address Construction (`NewETHAddr`)  
The `NewETHAddr` function creates a new unified address containing only an Ethereum address, setting the network address field to empty.  
  
#### Accessor Methods (`ETH`, `Addr`)  
The `ETH` method returns the Ethereum address stored in the struct (if present), returning an error if no Ethereum address is specified. The `Addr` method retrieves the network address (if set) and returns it; otherwise, it throws an error.  
  
#### String Representation (`String`, `MarshalText`)  
The `String` method provides a string representation of the unified address, combining the Ethereum and network addresses with "@" if both are present. The `MarshalText` method simply returns the result of calling `String()` as a byte slice.  
  
#### YAML Unmarshaling (`UnmarshalYAML`)  
The `UnmarshalYAML` function allows unmarshaling an `auth.Addr` object from YAML data. It parses the input string using `ParseAddr`, and if successful, assigns the parsed value to the receiver struct. If parsing fails, it returns a formatted error message.  
  
#### Error Handling (`errInvalidETHAddressFormat`)  
The `errInvalidETHAddressFormat` function creates an error indicating that the provided Ethereum address is in an invalid format.  
  
# insonmnia/auth/addr_test.go  
## Package/Component Summary: `auth`  
  
**Package Name:** `auth`  
  
**Imports:**  
*   `testing`: For unit testing functionality.  
*   `github.com/ethereum/go-ethereum/common`: Used for Ethereum address handling (specifically, converting hex strings to addresses).  
*   `github.com/stretchr/testify/assert`: Assertion library for writing tests.  
*   `github.com/stretchr/testify/require`: Requirement library for writing tests.  
  
**External Data / Input Sources:**  
The code primarily relies on string inputs representing Ethereum addresses and network endpoints (IP:Port). These strings are parsed by the `ParseAddr` function, which is not provided in this snippet but assumed to exist within the package. The test cases use hardcoded address/endpoint combinations for verification.  
  
**TODOs:** None found in the provided code.  
  
---  
  
### Code Summary Sections:  
  
**1. Address Parsing Tests (`TestNewAddr`, `TestNewAddrOnlyNet`, `TestNewAddrOnlyETH`)**:  
These tests verify that the `ParseAddr` function correctly parses valid address strings, including those with both Ethereum addresses and network endpoints, only network endpoints, or only Ethereum addresses. The parsed results are then checked against expected values using assertions from the `testify` library.  
  
**2. Error Handling Tests (`TestNewAddrErr`)**:  
This test confirms that `ParseAddr` returns an error when given invalid input strings (e.g., missing address part "@127.0.0.1:9090" or malformed addresses "WhatTheHell@127.0.0.1:9090"). The tests assert that the returned endpoint is `nil` and an error is present in such cases.  
  
**3. MarshalText Test (`TestAddrMarshalText`)**:  
This test verifies that the `MarshalText` method (presumably part of a struct returned by `ParseAddr`) correctly marshals the address into a byte slice, including prepending "0x" to the Ethereum address portion. The resulting byte slice is then compared against an expected value using assertions.  
  
# insonmnia/auth/auth.go  
**Package Name:** `auth`  
  
**Imports:**  
*   `context`: For managing request context.  
*   `math`: Used for maximum integer value in `expirationTimeFromDuration`.  
*   `sync`: Provides synchronization primitives (mutexes) for concurrent access to data structures.  
*   `time`: For handling time-related operations, such as expiration times and tickers.  
*   `github.com/ethereum/go-ethereum/common`: Used for Ethereum address representation (`common.Address`).  
*   `go.uber.org/zap`: Logging library.  
*   `google.golang.org/grpc/codes`: gRPC status codes (e.g., `Unauthenticated`).  
*   `google.golang.org/grpc/status`: For creating gRPC error responses.  
  
**External Data / Input Sources:**  
*   Context (`context.Context`) passed to authorization functions.  
*   Request interface (`interface{}`) which is the payload being authorized.  
*   Ethereum addresses (`common.Address`) for transport credential-based authorization.  
*   Configuration options (e.g., event prefixes, fallback authorizations) provided during router creation.  
  
**TODOs:** None found in this code snippet.  
  
---  
  
### Code Summary:  
  
**1. Core Authorization Structures & Interfaces:**  
  
The `AuthRouter` is the central component for gRPC authorization. It maintains a map of `Event` (gRPC method name) to `Authorization` implementations.  It allows registration of specific authorizations for events and provides a fallback mechanism (`fallback`) when no explicit rule exists. The `Authorization` interface defines an `Authorize` method that takes context and request data, returning an error if authorization fails.  
  
**2. Authorization Implementations:**  
  
*   `nilAuthorization`: Always allows access (no checks).  
*   `denyAuthorization`: Always denies access with a "permission denied" gRPC error.  
*   `transportCredentialsAuthorization`: Checks if the Ethereum address in the context matches a configured address (`ethAddr`).  Uses `FromContext` to extract peer info from the request context.  
*   `AnyOfTransportCredentialsAuthorization`: Allows dynamic addition and removal of authorized addresses with TTL (time-to-live). It uses a map of addresses to expiration times, expiring entries after their TTL.  
  
**3. Router Configuration Options:**  
  
The `EventAuthorizationOption` type allows configuring the router:  
  
*   `WithLog`: Sets a logger for debugging.  
*   `WithEventPrefix`: Adds a prefix to event names (useful for hierarchical authorization).  
*   `Allow`: Registers an authorization for specific events.  
*   `WithFallback`: Configures the default authorization when no rule matches.  
  
**4. Dynamic Authorization (`AnyOfTransportCredentialsAuthorization`)**:  
  
This component allows adding and removing authorized Ethereum addresses with a time-to-live (TTL). It uses a ticker to periodically check for expired entries, notifying subscribers via channels when an address is removed. The `Subscribe` method returns a channel that closes when the entry expires. This enables dynamic access control based on temporary credentials or permissions.  
  
**5. Utility Functions:**  
*   `equalAddresses`: Compares two Ethereum addresses for equality (not shown in snippet but likely used internally).  
*   `FromContext`: Extracts peer information from context, including Ethereum address (implementation not included here).  
  
# insonmnia/auth/common.go  
**Package Name:** `auth`  
  
**Imports:**  
*   `bytes`  
*   `context`  
*   `errors`  
*   `fmt`  
*   `net`  
*   `github.com/ethereum/go-ethereum/common`  
*   `google.golang.org/grpc/credentials`  
*   `google.golang.org/grpc/peer`  
  
**External Data / Input Sources:**  
*   Context (`context.Context`) for extracting peer information and wallet addresses.  
*   Network connections (`net.Conn`) for gRPC handshakes.  
*   Credentials (`credentials.TransportCredentials`) used as a base for custom authentication logic.  
*   Ethereum Addresses (`common.Address`) are central to the authentication process, including a hardcoded `LeakedInsecureKey`.  
  
**TODOs:**  
*   `// TODO: Left for backward compabitility, prune later.` - The function `equalAddresses` is marked as potentially removable in the future.  
  
---  
  
### Core Authentication Structures  
  
The code defines `EthAuthInfo`, which implements the `credentials.AuthInfo` interface and holds TLS information along with an Ethereum wallet address (`common.Address`). This structure represents a user's authentication credentials within the gRPC context. The `Peer` struct wraps `peer.Peer` from the grpc package, adding an associated Ethereum address for identification.  
  
### Context-Based Wallet Extraction  
  
The functions `FromContext` and `ExtractWalletFromContext` retrieve wallet addresses from the gRPC context (`context.Context`). They rely on the presence of `EthAuthInfo` within the peer information to extract the wallet. If no peer info or an unsupported auth type is found, errors are returned. This mechanism allows authentication based on pre-existing context data.  
  
### Wallet-Based Authentication Handlers  
  
The `WalletAuthenticator` struct wraps a base `credentials.TransportCredentials` and stores a target Ethereum address (`common.Address`). Its `ServerHandshake` and `ClientHandshake` methods perform gRPC handshakes, comparing the wallet address in the incoming authentication info against the stored target wallet. If they don't match, authorization fails. The comparison is done via `compareWallets`.  
  
### Address Comparison Utilities  
  
The functions `equalAddresses` (marked for future removal) and `EqualAddresses` compare two Ethereum addresses (`common.Address`) for equality using byte-level comparison of their underlying representations. This ensures that only connections from the expected wallet are authorized. The hardcoded insecure key is defined as a global variable, which could be used to bypass authentication if present in context.  
  
# insonmnia/auth/common_test.go  
## Package: `auth`  
  
**Imports:**  
  
*   `fmt`: For formatted I/O (specifically used in test failure messages).  
*   `testing`: Standard testing package for Go.  
*   `github.com/ethereum/go-ethereum/common`: Used for Ethereum address handling (`HexToAddress`).  
*   `github.com/stretchr/testify/assert`: Assertion library for writing tests.  
  
**External Data / Input Sources:**  
  
The code relies on string representations of hexadecimal addresses as input to the `TestEqualAddresses` function via test cases defined in a slice of structs. These strings are then converted into Ethereum addresses using `common.HexToAddress`. The expected equality results (`isEq`) for each address pair are also provided within these test cases.  
  
**TODOs:**  
  
There are no TODO comments present in the code.  
  
---  
  
### Test Function: `TestEqualAddresses`  
  
This function tests an internal (not shown) `equalAddresses` function, which presumably compares two Ethereum addresses for equality. The test suite uses a series of predefined address pairs (`cases`) with expected results to verify that the comparison logic works correctly under various conditions including different prefixes ("0x" vs no prefix), slight variations in hexadecimal values, and zero/non-zero addresses.  The `assert.Equal` function from testify is used to check if the actual result matches the expected equality flag for each case. The test cases cover scenarios with valid Ethereum address formats (with or without "0x" prefixes) as well as invalid ones ("0x").  
  
