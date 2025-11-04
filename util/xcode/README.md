# Package `xcode` Summary

This package implements a command-line tool for interacting with SONM services via gRPC using dynamically called methods and JSON payloads. It handles authentication through Ethereum keystores loaded from configuration files (default: `cli.yaml`). The core functionality revolves around dialing remote endpoints, marshaling/unmarshaling protobuf messages, executing service calls based on input parameters, and printing the response as JSON to standard output.

## Configuration & Inputs:

*   **`-remote` flag:** Required gRPC server address.
*   **`-input` flag:** Optional JSON file containing request body; defaults to an empty object if not provided.
*   **Ethereum Keystore/Passphrase:** Loaded from `cli.yaml` for authentication.  The keystore path is configurable via environment variables or command-line flags (not explicitly defined in this snippet).

## File Structure:

```
util/xcode/
├── cmd.go
```

## Key Functions & Logic:

*   **`init()`:** Initializes Cobra CLI, loads Ethereum keys from `cli.yaml`, and exits if loading fails.
*   **`Execute()`:** Executes the root command using Cobra.
*   **`RegisterServiceCmd()`:** Dynamically registers service commands for extension.
*   **`RunE()`:** Core logic: dials gRPC connection, retrieves method via reflection, marshals request from JSON (or empty object), calls service method, and prints response as JSON.
*   **`TypeToJson()`:** Generates a JSON template for a given protobuf type by instantiating an empty structure of that type and marshaling it into JSON format using `jsonpb`. Useful for creating request templates.
*   **`dial()`:** Establishes secure gRPC connection with TLS authentication using Ethereum keys.

## Edge Cases & Launching:

The application is launched via the command line using Cobra commands. The `-remote` flag must be provided, otherwise execution will fail. If no input file is specified (`-input`), an empty JSON object is used as the request body.  Key loading failures in `init()` cause immediate exit. No explicit error handling for invalid service method names or protobuf marshaling errors is shown in this snippet; these likely exist within the called gRPC methods themselves.