## Package: `disk`

This package retrieves free disk space information, primarily targeting the Docker root directory. It connects to the Docker daemon to determine the root path, falling back to the system root ("/") if Docker is unavailable. It uses `syscall.Statfs` to obtain disk space statistics.

**Configuration:**

*   **Environment Variables:** The Docker client is initialized using environment variables (presumably `DOCKER_HOST`, `DOCKER_TLS_VERIFY`, `DOCKER_CERT_PATH`, etc., though not explicitly defined in the code).
*   **Docker Daemon:** Requires a running Docker daemon to function optimally.

**Files:**

*   `insonmnia/hardware/disk/disk.go`

**Functions:**

*   `FreeDiskSpace()`: Retrieves disk space information.

**Data Structures:**

*   `Info`: Holds total and free disk space in bytes.

**Edge Cases:**

*   If the Docker daemon is unreachable or the Docker root directory cannot be determined, the function falls back to using the system root ("/").
*   Errors during `syscall.Statfs` will be propagated.

**Relations:**

The `FreeDiskSpace` function relies on the Docker client to determine the root directory. The `Info` struct is used to return the disk space statistics.