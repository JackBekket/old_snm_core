# insonmnia/ssh/config.go  
## Package: `ssh`  
  
**Imports:**  
  
*   `github.com/sonm-io/core/insonmnia/npp`  
  
**External Data/Input Sources:**  
  
*   Configuration is expected to be provided via YAML format, with fields `endpoint` (SSH server address) and `npp` (nested `npp.Config` structure). The `required:"true"` tags indicate mandatory fields.  
  
**TODOs:**  
  
*   None found in this snippet.  
  
**Code Summary:**  
  
### `ProxyServerConfig` Struct  
  
This struct defines the configuration for an SSH proxy server. It contains two fields:  
  
*   `Addr`: A string representing the SSH server's endpoint address. This field is required.  
*   `NPP`: An instance of the `npp.Config` struct from the `github.com/sonm-io/core/insonmnia/npp` package. This field is also required.  
  
The struct uses YAML tags (`yaml:"..."`) to map the fields to corresponding keys in a YAML configuration file.  
  
# insonmnia/ssh/crypto.go  
## Package: `ssh`  
  
**Imports:**  
  
*   `bytes`  
*   `crypto/ecdsa`  
*   `encoding/base32`  
*   `fmt`  
*   `strings`  
*   `github.com/btcsuite/btcd/chaincfg/chainhash`  
*   `github.com/ethereum/go-ethereum/common`  
*   `github.com/ethereum/go-ethereum/crypto`  
  
**External Data/Input Sources:**  
  
*   ECDSA private key (`*ecdsa.PrivateKey`) for identity creation.  
*   SSH identity string in the format "address@signature" for parsing.  
*   Ethereum address (`common.Address`) used for verification.  
*   Signature (`[]byte`) used for verification.  
  
**TODOs:**  
  
*   None found in the provided code.  
  
**Summary of Code Parts:**  
  
### SSH Identity Structure  
  
The `SSHIdentity` struct represents an SSH identity, containing an Ethereum address (`Addr`) and a signature (`Sign`). The `String()` method provides a string representation of the identity in the format "address@base32_encoded_signature".  
  
### Identity Creation (`NewSSHIdentity`)  
  
The `NewSSHIdentity` function creates a new `SSHIdentity` from an ECDSA private key. It derives the Ethereum address from the public key, calculates a double hash of the address, signs the hash using the private key, and returns a new `SSHIdentity` instance.  
  
### Identity Parsing (`ParseSSHIdentity`)  
  
The `ParseSSHIdentity` function parses an SSH identity string (in the format "address@signature") into an `SSHIdentity` struct. It decodes the base32-encoded signature and converts the address from hexadecimal format.  
  
### Identity Verification (`Verify`)  
  
The `Verify` function verifies the authenticity of an `SSHIdentity`. It recalculates the hash of the address, recovers the public key from the signature, and checks if the recovered public key corresponds to the provided Ethereum address. If the signature is invalid or doesn't match the address, it returns an error.  
  
# insonmnia/ssh/proxy.go  
## SSH Proxy Server Component Summary  
  
**Package Name:** `ssh`  
  
**Imports:**  
  
*   `context`  
*   `crypto/ecdsa`  
*   `fmt`  
*   `io`  
*   `math/big`  
*   `net`  
*   `os`  
*   `strings`  
*   `github.com/ethereum/go-ethereum/common`  
*   `github.com/gliderlabs/ssh` (aliased as `sshd`)  
*   `github.com/sonm-io/core/blockchain`  
*   `github.com/sonm-io/core/insonmnia/auth`  
*   `github.com/sonm-io/core/insonmnia/npp`  
*   `github.com/sonm-io/core/proto`  
*   `github.com/sonm-io/core/util/xgrpc`  
*   `go.uber.org/zap`  
*   `golang.org/x/crypto/ssh`  
*   `golang.org/x/crypto/ssh/agent`  
*   `golang.org/x/sync/errgroup`  
  
**External Data/Input Sources:**  
  
*   **SSH Agent:** Relies on a running SSH agent with loaded keys for authentication.  The `SSH_AUTH_SOCK` environment variable is used to locate the agent socket.  
*   **Blockchain Market API:**  Uses a `blockchain.MarketAPI` to resolve Deal IDs into Ethereum addresses.  
*   **NPP Dialer:** Uses `npp.Dialer` to establish connections to remote endpoints.  
*   **Configuration:** Takes `ProxyServerConfig` as input, including the address to listen on and NPP configuration.  
*   **Private Key:** Requires an `ecdsa.PrivateKey` for identity management.  
*   **User Input:** Extracts Deal ID and Task ID from the SSH username in the format `<DealID>.<TaskID>`.  
  
**TODOs:**  
  
*   "Activate relay, but for now disable for rendezvous testing."  
*   "stdout/stderr intermixing is possible. How to get with it?"  
  
**Code Sections Summary:**  
  
*   **SSH Proxy Server Implementation:** The `SSHProxyServer` struct and its methods (`NewSSHProxyServer`, `Serve`) implement the core SSH proxy functionality. It handles connection setup, authentication via SSH agent, and forwarding traffic to remote endpoints using NPP.  
*   **Connection Handling:** The `connHandler` struct and its methods (`onHandle`, `handle`, `extractMeta`, `resolve`) manage individual SSH connections. It extracts metadata from the session, resolves remote addresses using the blockchain market API, and establishes a tunnel to the remote endpoint.  
*   **User Identity Parsing:** The `parseUserIdentity` function parses the SSH username to extract the Deal ID and Task ID.  
*   **Utility Functions:** Functions like `formatErr` and `safeFingerprintSHA256` provide helper functionality for error formatting and fingerprint generation.  
*   **SSH Agent Integration:** The code interacts with the SSH agent to retrieve host signers for authentication.  
*   **NPP Integration:** The code uses NPP (Network Proxy Protocol) to dial remote endpoints.  
  
