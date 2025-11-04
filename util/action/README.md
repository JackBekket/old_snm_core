# Package `action`

The **`action`** package provides a lightweight framework for chaining and rolling back operations that implement the `Action` interface.  
It is split into two files:

```
util/action/
├── action.go
└── queue.go
```

## Overview of the code

| File | Purpose |
|------|---------|
| **action.go** | Declares the core `Action` interface and a helper function `Rollback`. |
| **queue.go** | Implements an `ActionQueue` that can be built from a variadic list of actions, executed sequentially, and rolled back on failure. |

### action.go

* **Interface `Action`**

  ```go
  type Action interface {
      Execute(ctx context.Context) error
      Rollback() error
  }
  ```

  * `Execute` performs the forward work.
  * `Rollback` undoes the operation.

* **Helper `Rollback(actions []Action)`**  
  Builds a simple stack (`deque`) from the supplied slice and repeatedly calls each action’s `Rollback()` method, aggregating any errors into a single multi‑error value.

### queue.go

* **Struct `ActionQueue`**

  ```go
  type ActionQueue struct {
      actions []Action
  }
  ```

  Holds the ordered list of actions to run.

* **Constructor**  
  ```go
  func NewActionQueue(actions ...Action) *ActionQueue { … }
  ```
  Creates a queue from a variadic argument list.

* **Method `(*ActionQueue).Execute`**

  ```go
  func (m *ActionQueue) Execute(ctx context.Context) (error, error) { … }
  ```

  Iterates over the queued actions, calling each `Execute`.  
  On success it returns `(nil, nil)`; on failure it returns an execution error and a rollback error.

* **Internal helper** – `deque` and its method `pop()` are used by `Rollback` to pop actions in reverse order.

## Environment variables / configuration

The package itself does not read any environment variable or command‑line flag.  
If you want to configure it externally, the following paths can be used:

* `util/action/queue.go` – contains all runtime logic; modify here if you need custom flags.
* `util/action/action.go` – contains the interface definition; change the signature of `Execute` or `Rollback` if you wish to pass additional parameters.

## Launching the application

If this package is used in a CLI main program, typical usage looks like:

```go
// In your main package:
import (
    "context"
    "github.com/sonm-io/core/util/action"
)

func main() {
    // Build actions (implementations of action.Action)
    q := action.NewActionQueue(myAction1, myAction2, myAction3)

    if err, rollbackErr := q.Execute(context.Background()); err != nil {
        log.Fatalf("execution failed: %v", err)
    } else if rollbackErr != nil {
        log.Printf("rollback errors: %v", rollbackErr)
    }
}
```

Edge cases:

* **Empty queue** – `NewActionQueue()` creates an empty queue; `Execute` will return `(nil, nil)` immediately.
* **Partial failure** – if any action’s `Execute` returns a non‑nil error, the method stops and returns that error along with any rollback errors collected so far.

---