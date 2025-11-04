# `<netutil>` – Network utilities package

## Overview  
`netutil` is a small Go library that simplifies working with network addresses, especially when parsing host‑port strings, resolving them to `net.TCPAddr`, and sorting IPs for display or further processing.

The package contains two source files:

```
util/netutil/
├── net.go
└── net_test.go
```

### Key features  
| Feature | What it does |
|---------|--------------|
| **Host‑port parsing** | `SplitHostPort`, `ExtractHost`, `ExtractPort` – split a string like `"::1:80"` into an IP and a port. |
| **YAML support** | `TCPAddr.UnmarshalYAML` – unmarshals a YAML string into a `net.TCPAddr`. |
| **IP discovery** | `GetAvailableIPs`, `GetPublicIPs` – collect all global unicast IPs from system interfaces, filter for public ones. |
| **Private‑IP detection** | `isPrivateIPv4`, `isPrivateIPv6`, `IsPrivateIP` – helpers that test whether an address is private (IPv4 or IPv6). |
| **TCP lookup** | `LookupTCPHostPort` – resolves a host‑port pair to a slice of `net.Addr`. |
| **Sorting** | `SortedIPs`, `sortableIPs` – convert string IPs into sorted strings, ordering IPv6 before IPv4. |

---

## Environment variables / flags / command‑line arguments  
The package itself does not expose any build tags or command‑line flags; it is intended to be imported by other code. However, the following are useful when using this library:

| Variable | Purpose |
|----------|---------|
| `netutil.HostPort` | A string like `"::1:80"` that can be passed to `SplitHostPort`. |
| `netutil.IPList` | Slice of IP strings used by `SortedIPs`. |

---

## How the code pieces relate  

* **Parsing** – `SplitHostPort` calls `net.ParseIP` and `strconv.Atoi` internally; it returns a `net.IP` and a custom type `Port uint16`.  
* **YAML unmarshalling** – `TCPAddr.UnmarshalYAML` uses the same parsing logic to populate its embedded `net.TCPAddr`.  
* **Discovery & filtering** – `GetAvailableIPs` walks all interfaces (`net.Interfaces()`), collects IPs, and feeds them into `GetPublicIPs`, which filters by `IsPrivateIP`.  
* **Lookup** – `LookupTCPHostPort` uses the parsed host‑port to perform a DNS lookup (`net.LookupTCPAddr`) for each interface.  
* **Sorting** – `SortedIPs` converts string IPs to `net.IP`, sorts them with `sortableIPs`, and returns a slice of strings again.

---

## Edge cases & launch scenarios  

1. **Multiple interfaces** – If the host has several network interfaces, `GetAvailableIPs` will return all global unicast addresses; `LookupTCPHostPort` will produce one `net.Addr` per interface.  
2. **IPv4 vs IPv6 ordering** – The custom sorter places IPv6 addresses before IPv4 ones (`Less` compares family first).  
3. **CLI usage** – If a main package imports `util/netutil`, it can call e.g.:

```go
ips := netutil.GetPublicIPs()
fmt.Println(netutil.SortedIPs(ips))
```

No special flags are required; the library is pure Go.

---

## File list (project structure)

```
util/
└─ netutil/
   ├─ net.go          // core implementation
   └─ net_test.go     // unit tests for parsing, detection, sorting
```

---  

**<end_of_output>**