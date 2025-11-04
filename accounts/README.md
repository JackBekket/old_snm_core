# Package `accounts`

The **`accounts`** package is a thin wrapper around an Ethereum keystore that lets you  
* load a single key,  
* manage multiple keys in a directory, and  
* persist the default account address.

It exposes two main abstractions:

| File | Purpose |
|------|---------|
| `config.go` | Holds the configuration struct (`EthConfig`) and loads a private key from a keystore file. |
| `multi_keystore.go` | Implements a *keystore manager* (`MultiKeystore`) that can list, generate, and retrieve keys; it also keeps track of the default account. |
| `single_keystore.go` | Provides low‑level helpers (`OpenSingleKeystore`, `getPassPhrase`, etc.) used by both `config.go` and `multi_keystore.go`. |
| `pass_reader.go` | Declares a `PassPhraser` interface and two concrete implementations (interactive & static). |
| `multi_keystore_test.go` | Unit tests for the manager. |

---

## Short summary of what the package does

* **Configuration** – `EthConfig` stores a keystore path and passphrase; it can be loaded from YAML or environment variables.
* **Key loading** – `LoadKey()` in `config.go` calls `OpenSingleKeystore()` to read an ECDSA key from the file specified by `EthConfig`.
* **Manager** – `MultiKeystore` keeps a directory of keystores, can list all accounts, generate new ones (optionally with a supplied password), and remember which account is “default”.
* **Passphrase handling** – The package supports two ways to obtain a passphrase: an interactive terminal prompt (`interactivePassPhraser`) or a static string (`staticPassPhraser`).  
  These are passed into the manager via the `PassPhraser` interface.

---

## Environment variables, flags and command‑line arguments that can be used for configuration

| Source | Variable / flag | Description |
|--------|-----------------|-------------|
| `EthConfig.Passphrase` | `--passphrase` (or env var `ETH_CONFIG_PASSPHRASE`) | Passphrase to decrypt the keystore. |
| `EthConfig.Keystore` | `--keystore` (or env var `ETH_CONFIG_KEYSTORE`) | Path of the keystore file. |
| `KeystoreConfig.KeyDir` | `--keydir` (or env var `KEY_DIR`) | Base directory that holds all key files and a state file. |
| `KeystoreConfig.PassPhrases` | `--passphrases` (map) | Optional map of passphrases keyed by address hex; used when generating keys with known passwords. |

The package itself does not expose any command‑line flags, but the public constructors (`NewMultiKeystore`, `NewInteractivePassPhraser`, etc.) can be called from a CLI program.

---

## Project package structure

```
accounts/
├── config.go
├── multi_keystore.go
├── multi_keystore_test.go
├── pass_reader.go
└── single_keystore.go
```

Each file is part of the same `accounts` package; all public types and functions are exported.

---

## Relations between code entities

* **EthConfig** (in `config.go`) → `LoadKey()` calls **OpenSingleKeystore()** (in `single_keystore.go`).  
  *The helper uses a `PassPhraser` supplied by the caller; in tests it is created with `NewInteractivePassPhraser()`.*

* **MultiKeystore** (in `multi_keystore.go`) holds:  
  - A `keystore.KeyStore` instance (`ks`).  
  - A `KeystoreConfig` (`cfg`).  
  - A `PassPhraser` (`pf`).  

  The constructor `NewMultiKeystore()` wires these together and calls **OpenSingleKeystore()** to initialise the keystore.

* **KeystoreConfig** provides helper methods `getStateFileDir()` / `getStateFilePath()` that locate a small state file inside `KeyDir/state/data`.  
  This file stores the hex address of the default account; it is read by `GetDefaultAddress()` and written by `setDefaultAccount()`.

* **PassPhraser** interface (in `pass_reader.go`) is implemented by:  
  - `interactivePassPhraser` – reads a passphrase from stdin/stdout.  
  - `staticPassPhraser` – returns a pre‑defined string.  

  Both are created via the exported constructors.

* **Key retrieval** – `GetKeyByAddress()` and `GetKeyWithPass()` in `multi_keystore.go` use the helper `readAccount()` to decrypt a key file; that helper internally calls `decryptKeyFile()` from `single_keystore.go`.

---

## Edge cases for launching an application

1. **Initialisation** – Call `NewMultiKeystore(NewKeystoreConfig(<dir>), NewInteractivePassPhraser())` or use the static reader if you already know the password.  
2. **Generating a key** – `Generate()` will create a new account and, if it is the first one, mark it as default.  
3. **Changing the default** – After generating several keys, call `SetDefault(<addr>)` to make any existing account the default; subsequent calls to `GetDefault()` will return that key.  
4. **Persisting state** – The state file (`<KeyDir>/state/data`) is created automatically if it does not exist; the package ensures the directory tree exists before writing.

---

## Summary of the whole package

*The `accounts` package provides a small but complete workflow for handling Ethereum keystores: load a single key, manage multiple keys in a directory, and remember which one is default.  It relies on the external `keystore` implementation from `github.com/ethereum/go-ethereum/accounts/keystore`, and it uses a simple YAML‑style configuration (`EthConfig`).  The public API consists of:*

| Function | Purpose |
|----------|---------|
| `NewMultiKeystore(cfg *KeystoreConfig, pf PassPhraser)` | Create a manager for a directory of keystores. |
| `(*MultiKeystore).List()` | Return all accounts in the keystore. |
| `(*MultiKeystore).GenerateWithPassword(pass string)` | Generate a new account with an explicit password. |
| `(*MultiKeystore).GetDefault()` | Load the default key from the state file. |
| `(*MultiKeystore).SetDefault(addr common.Address)` | Persist a chosen address as the default. |

*The tests in `multi_keystore_test.go` exercise these functions and confirm that the default account is updated correctly after each generation.*

---

**<end_of_output>**