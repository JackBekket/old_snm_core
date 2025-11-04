# xconcurrency Package Summary

This package provides a concurrent iteration utility. It allows processing elements of a slice or map concurrently using a specified number of goroutines. The core functionality is encapsulated in the `Run` function, which distributes elements to worker goroutines via a channel and ensures all workers complete before returning.

**Configuration:**

*   **`concurrency` (int):**  Determines the number of concurrent workers. Defaults to the length of the input iterable if smaller.
*   **`t` (interface{}):** The input iterable (slice or map).
*   **`cb` (func(elem interface{})):** The callback function executed for each element.

**Launch/Usage:**

The package is designed to be imported and used within other Go programs. The `Run` function is the primary entry point.  No direct command-line execution is available.

**Project Structure:**

```
util/
└── xconcurrency/
    └── cncurrency.go
```

**Relationships:**

The `Run` function is the central component. It leverages `reflect` to handle slices and maps generically, `sync.WaitGroup` for synchronization, and goroutines for concurrency. The callback function `cb` is passed as an argument, allowing external logic to be applied to each element.

**Edge Cases:**

*   If `concurrency` is greater than the length of the input slice/map, it's reduced to the iterable's length.
*   The function panics if the input `t` is neither a slice nor a map.