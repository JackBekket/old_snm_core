## Package: `generate_api` (Solidity Contract Wrapper Generator)

This package generates Go bindings for Solidity contracts based on their artifacts. It reads JSON artifacts from a specified directory, filters out unwanted contracts (e.g., `IterableMapping`), and uses the `go-ethereum/accounts/abi/bind` package to create Go wrappers. The generated wrappers are written to individual `.go` files in a designated output directory.

**Configuration:**

*   `solidityArtifactsPath`: Path to the directory containing Solidity artifact JSON files (default: `./build/contracts/`).
*   `wrappersPath`: Path to the directory where generated Go wrappers will be written (default: `./api`).
*   `wrappersPackage`: The Go package name for the generated wrappers (default: `api`).

**Environment Variables/Flags/Cmdline Arguments:**

None explicitly defined in the provided code. Configuration is hardcoded.

**Files and Paths:**

*   `./build/contracts/*.json`: Input Solidity artifact files.
*   `./api/*.go`: Output Go wrapper files.

**Edge Cases:**

*   If the `./build/contracts/` directory does not exist or contains no valid JSON files, the program will exit with an error.
*   The `IterableMapping.json` file is explicitly skipped.
*   File permissions for generated wrappers are set to 0600.

**Project Package Structure:**

```
blockchain/
└── source/
    └── utils/
        └── generate_api.go
```

**Relations Between Code Entities:**

The `generate_api.go` file contains the main logic for reading Solidity artifacts, generating Go bindings, and writing the output files. The `SolidityArtifact` struct represents the structure of the input JSON files. The `github.com/ethereum/go-ethereum/accounts/abi/bind` package is used to perform the actual binding generation. The `dieSoon` function provides a centralized error handling mechanism.