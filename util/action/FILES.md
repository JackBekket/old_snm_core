# util/action/action.go  
# Package: `action`  
  
## Imports    
- `context` – standard library package used for passing execution context to actions.  
  
## External data / input sources    
The interface relies on the `context.Context` type from the Go standard library, which allows callers to provide cancellation signals and deadlines when executing an action.  
  
## TODOs    
No explicit TODO comments are present in this file.  
  
## Summary of major code parts    
  
### Interface `Action`  
- **Purpose**: Defines a contract for any operation that can be executed and later rolled back.    
- **Methods**    
  - `Execute(ctx context.Context) error` – performs the action’s primary work, returning an error if something goes wrong.    
  - `Rollback() error` – undoes the action, also returning an error on failure.    
  
This interface is intended to be implemented by concrete types that encapsulate a unit of work (e.g., database migration, file manipulation, etc.) and provide both forward and reverse operations.  
  
# util/action/queue.go  
**Package / Component name**    
`action`  
  
---  
  
### Imports  
  
| Package | Purpose |  
|---------|---------|  
| `context` | Provides the `Context` interface used in `Execute`. |  
| `github.com/sonm-io/core/util/multierror` | Supplies a multi‑error helper for aggregating rollback errors. |  
  
---  
  
### External data / input sources    
* The queue is built from a variadic list of `Action` objects (`NewActionQueue`).    
* Execution receives a `context.Context` value and returns two error values: the first one represents an execution failure, the second aggregates any rollback errors.  
  
---  
  
### TODO comments    
No explicit TODO markers are present in this file.  
  
---  
  
## Major code parts  
  
#### 1. `ActionQueue` struct  
```go  
type ActionQueue struct {  
	actions []Action  
}  
```  
* Holds a slice of actions that will be executed sequentially.  
  
#### 2. `NewActionQueue` constructor  
```go  
func NewActionQueue(actions ...Action) *ActionQueue { … }  
```  
* Creates an `ActionQueue` instance from a variadic list of actions.  
  
#### 3. `(*ActionQueue).Execute` method  
```go  
func (m *ActionQueue) Execute(ctx context.Context) (error, error) { … }  
```  
* Iterates over the queued actions, executing each one.  
* On failure it immediately returns the execution error and a rollback of all previously executed actions.  
* On success it returns `nil, nil` (the second value is intended to hold rollback errors).  
  
#### 4. `Rollback` helper  
```go  
func Rollback(actions []Action) error { … }  
```  
* Builds an internal deque from the given slice and repeatedly pops actions, invoking their `Rollback()` method.  
* Aggregates all rollback errors into a single multi‑error value.  
  
#### 5. `deque` struct  
```go  
type deque struct {  
	actions []Action  
}  
```  
* A lightweight stack used by `Rollback`.  
  
#### 6. `(*deque).pop` method  
```go  
func (m *deque) pop() (Action, bool) { … }  
```  
* Pops the last action from the internal slice and returns it together with a success flag.  
  
---  
  
