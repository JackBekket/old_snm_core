```markdown
## rest Package Summary

This package implements a RESTful interface over gRPC services, providing functionality for decoding incoming requests (potentially encrypted via AES), encoding outgoing responses, and integrating with structured logging using Zap. It leverages functional options to configure server behavior, including custom decoders/encoders and gRPC interceptors. The core logic revolves around mapping HTTP endpoints to underlying gRPC methods through reflection and JSON serialization/deserialization.

**Configuration:**

*   **Environment Variables:** None explicitly defined in the code.
*   **Flags/Cmdline Arguments:** Not applicable (this is a library package, not an executable).
*   **Files & Paths:** Configuration is primarily done via functional options passed to `NewServer`. The `options.go` file defines these options:
    *   `WithLog`: Sets the Zap logger instance.
    *   `WithDecoder`: Configures the request body decoder (defaults to no decoding).
    *   `WithEncoder`: Configures the response encoder (defaults to no encoding).
    *   `WithInterceptors`: Applies gRPC unary server interceptors, including a default Zap logging interceptor.

**Edge Cases/Launch:**

This is not an executable package; it's intended for integration into larger applications. Launching involves instantiating `NewServer`, configuring options as needed, registering services via `RegisterService`, and then starting the HTTP server using `Serve`. The TODO comment in `server.go` indicates that streaming RPC methods are currently unsupported.

**Project Package Structure:**

```
util/rest/
├── aes.go        # AES encryption/decryption for request/response bodies.
├── errors.go     # Conversion between gRPC and HTTP error codes.
├── options.go    # Functional options for configuring the server (logging, decoders, encoders).
└── server.go     # Main REST server implementation with service registration and HTTP handling.
```

**Code Relations & Unclear Places:**

*   The `AESDecoderEncoder` in `aes.go` suggests a security layer where request/response bodies can be encrypted using AES. The encryption key is provided during initialization, making it critical for secure communication.
*   The integration of gRPC interceptors via `options.go` and `server.go` allows applying middleware to gRPC calls (e.g., logging). This suggests a hybrid architecture where REST endpoints proxy to underlying gRPC services.
*   The use of reflection in `server.go` for method invocation is powerful but can introduce runtime errors if service interfaces change without corresponding updates. The TODO comment about streaming methods indicates an incomplete feature set.

<end_of_output>
```