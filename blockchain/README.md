Okay, here's a markdown summary of the provided blockchain package code, following your instructions.

# Blockchain Package Summary

This package implements core blockchain interaction logic, likely for a decentralized application (dApp) built on Ethereum or a compatible chain. It provides interfaces and concrete implementations for interacting with smart contracts, managing transactions, and handling events. The package heavily relies on `go-ethereum` for low-level blockchain access.

**Configuration:**

*   **Endpoints:** Masterchain and sidechain endpoints are configurable, with hardcoded defaults (Infura and a SONM sidechain).
*   **Gas Prices:** Gas prices are configurable, with default values for both chains.
*   **Contract Registry:** The contract registry address is configurable.
*   **Batch Size:** The number of blocks processed in batches is configurable.
*   **YAML Configuration:** The package supports loading configuration from a YAML file.

**Environment Variables/Cmdline Arguments:**

The package doesn't explicitly define environment variables or command-line arguments. Configuration is loaded from YAML or hardcoded defaults.

**Edge Cases (Launch):**

The package is designed to be integrated into a larger application. It doesn't have a standalone launch point. The `client.go` file suggests that the package is intended to be used with a custom Ethereum client implementation.

**Project Package Structure:**

```
blockchain/
├── addresses.go
├── api.go
├── api_ext.go
├── api_ext_test.go
├── client.go
├── client_test.go
├── config.go
├── gasLimits.go
├── options.go
├── source/
│   ├── .babelrc
│   ├── .eslint
│   ├── .eslintignore
│   ├── .eslintrc
│   ├── .gitignore
│   ├── .solcover.js
│   ├── .soliumignore
│   ├── .soliumrc.json
│   ├── Makefile
│   ├── api/
│   │   ├── AddressHashMap.go
│   │   ├── BasicToken.go
│   │   ├── Blacklist.go
│   │   ├── DeployList.go
│   │   ├── DevicesStorage.go
│   │   ├── ERC20.go
│   │   ├── ERC20Basic.go
│   │   ├── Market.go
│   │   ├── Migrations.go
│   │   ├── MultiSigWallet.go
│   │   ├── OracleUSD.go
│   │   ├── Ownable.go
│   │   ├── Pausable.go
│   │   ├── ProfileRegistry.go
│   │   ├── SNM.go
│   │   ├── SNMMasterchain.go
│   │   ├── SafeMath.go
│   │   ├── SimpleGatekeeperWithLimit.go
│   │   ├── SimpleGatekeeperWithLimitLive.go
│   │   ├── StandardToken.go
│   │   └── TestnetFaucet.go
│   ├── contracts/
│   │   ├── AddressHashMap.sol
│   │   ├── Administratable.sol
│   │   ├── AutoPayout.sol
│   │   ├── Blacklist.sol
│   │   ├── DeployList.sol
│   │   ├── DevicesStorage.sol
│   │   ├── Market.sol
│   │   ├── Migrations.sol
│   │   ├── MultiSigWallet.sol
│   │   ├── OracleUSD.sol
│   │   ├── ProfileRegistry.sol
│   │   ├── SNM.sol
│   │   ├── SNMMasterchain.sol
│   │   ├── SimpleGatekeeperWithLimit.sol
│   │   ├── SimpleGatekeeperWithLimitLive.sol
│   │   └── TestnetFaucet.sol
│   ├── deployed/
│   │   ├── AddressHashMap.json
│   │   ├── BasicToken.json
│   │   ├── Blacklist.json
│   │   ├── DeployList.json
│   │   ├── ERC20.json
│   │   ├── ERC20Basic.json
│   │   ├── Market.json
│   │   ├── Migrations.json
│   │   ├── MultiSigWallet.json
│   │   ├── OracleUSD.json
│   │   ├── Ownable.json
│   │   ├── Pausable.json
│   │   ├── ProfileRegistry.json
│   │   ├── SNM.json
│   │   ├── SNMMasterchain.json
│   │   ├── SafeMath.json
│   │   ├── SimpleGatekeeper.json
│   │   ├── StandardToken.json
│   │   └── TestnetFaucet.json
│   ├── migration_artifacts/
│   │   ├── AddressHashMap.json
│   │   ├── Administratable.json
│   │   ├── AutoPayout.json
│   │   ├── BasicToken.json
│   │   ├── BasicTokenDeployed.json
│   │   ├── Blacklist.json
│   │   ├── DeployList.json
│   │   ├── DevicesStorage.json
│   │   ├── Dummy.json
│   │   ├── ERC20.json
│   │   ├── ERC20Basic.json
│   │   ├── ERC20BasicDeployed.json
│   │   ├── ERC20Deployed.json
│   │   ├── Market.json
│   │   ├── Migrations.json
│   │   ├── MultiSigWallet.json
│   │   ├── OracleUSD.json
│   │   ├── Ownable.json
│   │   ├── Pausable.json
│   │   ├── ProfileRegistry.json
│   │   ├── SNM.json
│   │   ├── SNMMasterchain.json
│   │   ├── SafeMath.json
│   │   ├── SimpleGatekeeperWithLimit.json
│   │   ├── SimpleGatekeeperWithLimitLive.json
│   │   ├── StandardToken.json
│   │   └── StandardTokenDeployed.json
│   ├── migration_utils/
│   │   ├── address_hashmap.js
│   │   └── multisig.js
│   ├── migrations/
│   │   ├── 1_initial_migration.js
│   │   ├── 2_deploy_v1.js
│   │   ├── 3_deploy_devices_storage.js
│   ├── package-lock.json
│   ├── package.json
│   ├── patches/
│   │   ├── truffle+4.1.14.patch
│   │   └── truffle-resolver+4.0.4.patch
│   ├── scripts/
│   │   ├── dev.sh
│   │   ├── test.sh
│   │   └── test_coverage.sh
│   ├── test/
│   │   ├── Administratable.js
│   │   ├── addressHashMap.js
│   │   ├── autoPayout.js
│   │   ├── blacklist.js
│   │   ├── deployers.js
│   │   ├── devicesStorage.js
│   │   ├── helpers/
│   │   │   ├── EVMRevert.js
│   │   │   ├── EVMThrow.js
│   │   │   ├── advanceToBlock.js
│   │   │   ├── ask.js
│   │   │   ├── common.js
│   │   │   ├── constants.js
│   │   │   ├── decodeLogs.js
│   │   │   ├── ether.js
│   │   │   ├── expectEvent.js
│   │   │   ├── expectThrow.js
│   │   │   ├── hashMessage.js
│   │   │   ├── increaseTime.js
│   │   │   ├── latestTime.js
│   │   │   ├── merkleTree.js
│   │   │   ├── printers.js
│   │   │   └── toPromise.js
│   │   ├── market.js
│   │   ├── multiSigWallet.js
│   │   ├── oracleUSD.js
│   │   ├── profileRegistry.js
│   │   ├── simpleGatekeeperWithLimit.js
│   │   ├── simpleGatekeeperWithLimitLive.js
│   ├── truffle.js
│   ├── utils/
│   │   └── generate_api.go
│   └── topics.go
├── types.go
├── types_test.go
├── unit.go
└── util.go
```

**Relations Between Code Entities:**

*   **`api.go`:** Defines interfaces for interacting with blockchain components (Market, Profile Registry, Blacklist, etc.).
*   **`api_ext.go`:** Provides a concrete implementation (`niceMarketAPI`) that enhances the `MarketAPI` with profile and blacklist checks.
*   **`client.go`:** Handles low-level Ethereum client interaction (fetching blocks, receipts, balances).
*   **`config.go`:** Manages configuration parameters (endpoints, gas prices, contract registry).
*   **`gasLimits.go`:** Defines gas limits for various operations.
*   **`source/contracts/*.sol`:** Solidity smart contracts for the dApp's core logic.
*   **`source/migrations/*.js`:** Truffle migration scripts for deploying contracts.
*   **`source/test/*.js`:** JavaScript tests for smart contracts.

**Unclear Places/Dead Code:**

*   The `// TODO: Here the market bug, but we need to live with it #1293.` comment in `api_ext.go` indicates a known, unaddressed bug in the market logic.
*   The `source/patches` directory suggests that the project relies on custom patches for Truffle, which could introduce instability or compatibility issues.
*   The `source/scripts` directory contains shell scripts for development and testing, but their exact purpose is unclear without further context.
*   The `source/migration_artifacts` directory contains compiled contract artifacts, which are likely generated during the deployment process.

The package appears to be a complex system for interacting with a blockchain-based dApp, with a focus on market operations, identity management, and security (blacklists, multi-sig). The presence of known bugs and custom patches suggests that the project may be in an unstable or unfinished state.