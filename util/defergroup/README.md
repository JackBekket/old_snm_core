# Package `defergroup`

## Short summary  
`util/defergroup/mod.go` implements a lightweight helper that mimics Go’s `defer` statement but allows the caller to cancel execution and run cleanup functions in reverse order. It is intended for use inside constructors or other code paths where multiple resources are allocated and must be released on success.

---

## Environment variables, flags & command‑line arguments  
*No external configuration items are defined; all behaviour is driven by the exported type and its methods.*

---

## Project package structure  

```
util/
└─ defergroup/
   └─ mod.go
```

---

## How the code works

| Entity | Purpose | Key details |
|--------|---------|-------------|
| `type DeferGroup struct { fn []func(); canceled bool }` | Holds a slice of cleanup callbacks and a flag that indicates whether execution has been cancelled. | The slice is appended to by `Defer`; `Exec` runs the functions in reverse order; `CancelExec` simply flips the flag. |
| `func (m *DeferGroup) Defer(fn func())` | Adds a new function to the list of deferred actions. | Appends `fn` to `m.fn`. |
| `func (m *DeferGroup) Exec()` | Executes all stored functions in reverse order, mirroring Go’s `defer`. | Iterates over indices from `len(m.fn)-1` down to `0`, calling each function. |
| `func (m *DeferGroup) CancelExec()` | Marks the group as cancelled so that subsequent calls to `Exec` will be no‑ops. | Sets `m.canceled = true`. |

### Typical usage pattern

```go
dg := DeferGroup{}
defer dg.Exec()          // ensure cleanup runs when the surrounding function returns

resource, err := Allocate()
if err != nil {
    return err
}
dg.Defer(func() { resource.Close() })

// …more allocations…

dg.CancelExec()          // everything succeeded – no cleanup needed
return nil
```

The pattern mirrors Go’s `defer` but gives the caller explicit control over when the cleanup functions are executed and whether they should be cancelled.

---

## Edge cases for launching an application  
If this package is used in a command‑line tool, it can be instantiated at the very beginning of `main()`:

```go
func main() {
    dg := util.DeferGroup{}
    defer dg.Exec()
    // …
}
```

Because `Exec` runs on return, any early error will still trigger cleanup. The only edge case to watch for is that `CancelExec` must be called before the function returns; otherwise the cleanup functions are executed twice (once by `defer` and once by an explicit call).

---

## Summary of logic  
* `DeferGroup` collects callbacks.  
* `Defer` appends a callback.  
* `Exec` runs them in reverse order, ensuring that resources are released in the opposite order they were allocated.  
* `CancelExec` simply toggles a flag to avoid double execution.

No dead code or missing references were detected; all methods are used by the example usage pattern.