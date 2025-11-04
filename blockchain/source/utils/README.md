# main

## Overview  
`blockchain/source/utils/generate_api.go` is a small command‑line tool that scans Truffle‑compiled Solidity artifacts, generates Go bindings for each contract with the `bind.Bind` helper from the Ethereum Go SDK, and writes the resulting wrapper files into an `api/` directory. The generated code lives in the package named **api**, while this utility itself resides in the **main** package.

---

## File structure  

```
blockchain/
└─ source/
   └─ utils/
      └─ generate_api.go
```

*Only one file is present; all logic is contained here.*

---

## Environment / configuration

| Variable | Purpose | Default value |
|----------|---------|---------------|
| `solidityArtifactsPath` | Glob pattern for Truffle JSON artifacts | `"./build/contracts/*.json"` |
| `wrappersPath` | Output directory for generated Go files | `"api"` |
| `wrappersPackage` | Package name used when calling `bind.Bind` | `"api"` |

These constants can be tweaked to point at a different build folder or change the output package.

---

## How it works

1. **Imports** – pulls in standard packages plus `github.com/ethereum/go-ethereum/accounts/abi/bind`.
2. **Constants** – define paths and package name.
3. **`SolidityArtifact` struct** – matches the JSON structure produced by Truffle (`contractName`, `abi`, `bytecode`).  
4. **Helper `dieSoon(e error, msg string)`** – prints an error message and exits if a step fails.
5. **`main()`** –  
   * Uses `filepath.Glob(solidityArtifactsPath)` to find all artifact files.  
   * Creates the output directory (`os.MkdirAll`).  
   * Loops over each JSON file:  
     - Skips `"build/contracts/IterableMapping.json"` (hard‑coded).  
     - Reads, unmarshals into `SolidityArtifact`.  
     - Marshals the ABI back to bytes for `bind.Bind`.  
     - Calls `bind.Bind` with the contract name, ABI, bytecode and package name.  
     - Writes the resulting Go source to `api/<ContractName>.go`.

After the loop finishes, a set of wrapper files is ready for import elsewhere in the project.

---

## Launch edge‑cases

* **Build** – `go build -o generate_api ./blockchain/source/utils` creates an executable that can be run directly.  
* **Run** – `go run ./blockchain/source/utils/generate_api.go` will perform the same steps without producing a binary.  
* **Re‑generation** – Running again will overwrite existing wrapper files; no deduplication logic is present, so manual cleanup may be needed if artifacts change.

---

## Summary of major code parts

| Section | Purpose |
|---------|---------|
| Constants | Define artifact glob, output dir & package name. |
| `SolidityArtifact` struct | Holds contract metadata for JSON unmarshalling. |
| `dieSoon` helper | Simple error handling wrapper. |
| `main()` loop | Core logic: discover artifacts → generate Go bindings → write files. |

No TODO comments were found; the code appears complete and self‑contained.

---