# insonmnia/hardware/disk/disk.go  
## Package: `disk`  
  
**Imports:**  
  
*   `context`  
*   `fmt`  
*   `syscall`  
*   `github.com/docker/docker/client`  
  
**External Data/Input Sources:**  
  
*   Docker root directory path obtained from Docker client info.  
*   System root directory ("/") as a fallback if Docker root cannot be stat'ed.  
*   Docker client is initialized using environment variables.  
  
**TODOs:**  
  
*   None  
  
---  
  
### `Info` Struct  
  
Defines a struct `Info` containing `TotalBytes` and `FreeBytes` as `uint64` representing disk space information.  
  
### `FreeDiskSpace` Function  
  
This function retrieves free disk space for the Docker root path. It first attempts to connect to the Docker daemon using `client.NewEnvClient()`. If successful, it retrieves Docker info to determine the root directory. It then uses `syscall.Statfs` to get disk space statistics. If the Docker root cannot be stat'ed, it falls back to using the system root ("/"). The function returns a pointer to an `Info` struct containing total and free bytes, or an error if any step fails. The Docker client is closed using `defer cli.Close()`.  
  
