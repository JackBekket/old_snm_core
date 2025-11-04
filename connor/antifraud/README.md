## Anti-Fraud Package Summary

**Package Name:** `antifraud`

This package implements an anti-fraud system within the SONM ecosystem, monitoring deals (tasks) for quality and blacklisting suppliers if they consistently provide low-quality work. It leverages metrics collection via Prometheus, structured logging with Zap, gRPC communication with other services, and configurable processing pipelines to analyze task execution data. The core logic revolves around tracking deal activity, extracting hashrate from logs or pool APIs, comparing performance against benchmarks, and dynamically adjusting blacklisting status based on observed behavior.

**Configuration:**

The package relies heavily on configuration loaded from YAML files via the `Config` struct. Key parameters include:
*   `TaskQuality`: Threshold for acceptable task quality (float64). Deals below this threshold may be closed with blacklisting.
*   `TrackInterval`: Frequency of checking deal status and updating metrics.
*   `WarmupDelay`: Initial delay before quality checks begin, allowing tasks to stabilize.
*   `BlacklistSettings`: Configuration for blacklist management, including timeouts and retry policies.
*   `WhitelistAddresses`: List of Ethereum addresses exempt from blacklisting.

**Environment Variables/Flags:** None explicitly defined in the provided code snippets; configuration is primarily file-based. The `flags.go` file defines a custom type with methods to control certain checks (e.g., skipping blacklisting).

**File Structure:**

*   `antifraud.go`: Main entry point and core logic for anti-fraud processing.
*   `blacklist_watcher.go`: Manages blacklist status, dynamically adjusting check frequency based on supplier behavior.
*   `config.go`: Defines configuration structures loaded from YAML files.
*   `flags.go`: Custom type with methods to control certain checks (e.g., skipping blacklisting).
*   `log_processor.go`: Extracts hashrate data from task logs, analyzes performance against benchmarks.
*   `pool_processor.go`: Fetches worker hashrates from mining pool APIs (Dwarfpool, UleyPool), calculates quality metrics.
*   `processor.go`: Defines the `Processor` interface and factory for creating log/pool processors based on configuration.

**Edge Cases:**

The package handles potential errors gracefully through retries, timeouts, and logging. The blacklist watcher dynamically adjusts check frequency to avoid excessive API calls or false positives. Configuration validation ensures that critical parameters (e.g., decay times) are within acceptable ranges. However, the code relies heavily on external dependencies (gRPC clients, pool APIs), which could introduce failure points if those services become unavailable.

**Potential Issues:**

*   The TODO comments in `antifraud.go` suggest incomplete features: asynchronous deal checking and database persistence for blacklist data.
*   The reliance on hardcoded whitelist addresses in tests may not reflect production configurations.
*   The lack of explicit error handling for external API failures could lead to unexpected behavior if pool APIs are unreliable.