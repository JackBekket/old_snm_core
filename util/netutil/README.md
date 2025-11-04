```markdown
# netutil

## Summary

The `netutil` package provides utility functions for network address manipulation, parsing, validation, and sorting. It includes functionality to extract host/port from strings, unmarshal TCP addresses from YAML, retrieve public IP addresses, check if an IP is private, resolve host:port combinations, and sort IP lists with prioritization of public IPs.

## Configuration & Arguments

The package relies on system network interfaces for retrieving available IPs. No explicit configuration files or command-line arguments are present in the provided code snippets. The behavior can be influenced by the underlying operating system's networking setup.

## Package Structure

```
util/netutil/
├── net.go
└── net_test.go
```

## Core Components & Relations

*   **`SplitHostPort`, `ExtractHost`, `ExtractPort`**: These functions parse host:port strings, similar to the standard library but with IPv6 bracket handling. They are foundational for other operations that require separating address components.
*   **`TCPAddr`**: A custom type enabling YAML unmarshalling of TCP addresses. This suggests integration with configuration systems using YAML format.
*   **`GetPublicIPs`, `GetAvailableIPs`, `IsPrivateIP`**: These functions work together to identify usable public IP addresses from network interfaces, filtering out private or invalid entries.  The `IsPrivateIP` function relies on hardcoded IPv4/IPv6 ranges for detection.
*   **`LookupTCPHostPort`**: Resolves host:port strings into a slice of `net.Addr`, using DNS lookup via `net.LookupHost`. This is useful for connecting to remote services by hostname.
*   **`SortedIPs`**: Sorts IP addresses, prioritizing public IPs (IPv6 before IPv4) over private ones. The sorting logic might be crucial in scenarios where specific address order matters (e.g., load balancing).

## Edge Cases & Launching

The package is a utility library and doesn't have a direct entry point for launching as an application. It's intended to be imported into other Go programs that require network-related functionality. The behavior depends on how the functions are called within those applications. No specific edge cases related to command-line arguments or startup flags exist, since it is not executable.

## Unclear Places & Dead Code

The provided code snippets do not reveal any obvious dead code or unclear places. However, without seeing the full implementation of `isPrivateIPv4` and `isPrivateIPv6`, their exact behavior cannot be fully determined. The test cases in `net_test.go` suggest that these functions are well-tested but their internal logic remains hidden.

<end_of_output>
```