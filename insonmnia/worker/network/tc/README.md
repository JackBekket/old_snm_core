# Package `tc`

**Short summary**  
The `tc` package implements a lightweight abstraction over Linux traffic‑control (`tc`) and the Netlink API.  It defines data structures for queueing disciplines, classes, and filters; provides constructors for both a command‑based implementation (using the external `tc` binary) and a native Netlink implementation (Linux only).  The package also contains helper types for handles, protocols, and ifb module management.

---

## Environment variables, flags & command‑line arguments

| Variable / Flag | Purpose |
|------------------|---------|
| `TCActionAdd`, `TCActionDel` | Action strings used when building sub‑commands (`add`/`del`). |
| Build tags | `// +build linux,nl` – Linux‑specific implementation; `// +build !linux !nl` – generic fallback. |

The package can be invoked via the exported interface `TC`, e.g.:

```go
tc, err := tc.NewDefaultTC()   // or NewNetlinkTC on Linux
err = tc.QDiscAdd(myPfifo)       // add a queueing discipline
```

---

## File structure

```
insonmnia/worker/network/tc/
├── class.go
├── filter.go
├── filter_test.go
├── handle.go
├── handle_test.go
├── ifb.go
├── proto.go
├── proto_linux.go
├── proto_nonlinux.go
├── qdisc.go
├── qdisc_test.go
├── tc.go
├── tc_linux.go
└── tc_nonlinux.go
```

---

## Key code entities & their relations

| Entity | Purpose | Relations |
|--------|---------|------------|
| `Handle` (handle.go) | 32‑bit identifier for classes, qdiscs, filters. | Used by `ClassAttrs`, `FilterAttrs`, `QDiscAttrs`. |
| `ProtoAll`, `ProtoIP` (proto*.go) | Protocol constants for filter matching. | Referenced in `filter.go`. |
| `ClassAttrs` (class.go) | Holds link, handle, parent for a class. | Embedded in concrete classes (`HTBClass`). |
| `HTBClass` (class.go) | HTB traffic‑control class with rate/ceil. | Implements `Class` interface; used by `tc_linux.go`. |
| `FilterAttrs` (filter.go) | Common attributes for filters. | Embedded in `U32`. |
| `U32Key`, `U32` (filter.go) | 32‑bit classifier filter and its key. | Used by `tc_linux.go`. |
| `QDiscAttrs` (qdisc.go) | Common attributes for queueing disciplines. | Embedded in concrete qdiscs (`PfifoQDisc`, `TFBQDisc`, `HTBQDisc`). |
| Concrete qdiscs (`PfifoQDisc`, etc.) | Implement specific queueing disciplines. | Each implements `QDisc`; used by `tc_linux.go`. |
| `TC` interface (tc.go) | Public API for adding/deleting qdiscs, classes, filters. | Implemented by `CmdTC` and `NetlinkTC`. |
| `NewCmdTC`, `NewNetlinkTC` (tc*.go) | Constructors for the two implementations. | `NewDefaultTC` forwards to one of them. |
| `execModProbe` (ifb.go) | Helper that runs `modprobe`. | Used by `IFBInit/Close/Flush`. |

---

## Edge cases / launch scenarios

1. **Linux build** – `tc_linux.go` is compiled when the tags `linux,nl` are set.  
   * `NewNetlinkTC()` creates a Netlink socket and exposes methods `QDiscAdd`, `ClassAdd`, `FilterAdd`.  
   * The package can be used as a library or wrapped in a higher‑level CLI that calls these methods.

2. **Non‑Linux build** – `tc_nonlinux.go` provides the fallback implementation using the external `tc` binary.  
   * `NewCmdTC()` locates the binary and executes commands via `exec.Command`.  

3. **Command execution** – The exported interface allows a caller to:
   ```go
   tc, _ := tc.NewDefaultTC()
   pfifo := &PfifoQDisc{...}
   err = tc.QDiscAdd(pfifo)
   ```
   * The helper functions in `tc_linux.go` build the appropriate Netlink structures and invoke libnl3 calls.

---

## Summary of logic

* **Handles** – thin wrapper around a 32‑bit value; helpers for string conversion, minor adjustment.  
* **Protocols** – simple enum with stringer.  
* **Classes / Filters / QDiscs** – each type embeds its attribute struct and implements the `Cmd()` method that returns a slice of strings ready to be passed to the underlying command or Netlink call.  
* **Ifb module** – provides init/close/flush helpers for the kernel module.  
* **tc.go** – defines the public interface and two concrete implementations; the Linux version uses cgo to talk directly to libnl3, while the non‑Linux version simply runs the external `tc` binary.

---