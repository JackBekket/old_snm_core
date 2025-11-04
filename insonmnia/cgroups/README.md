# Package `cgroups`

A lightweight abstraction over the **containerd** cgroup API that lets you create, manage and query nested control groups in a single package.  
The implementation is split into three files:

| Path | Purpose |
|------|---------|
| `insonmnia/cgroups/cgroup.go` | Core interface definitions (`CGroup`, `CGroupManager`) and the generic manager logic that works on any platform. |
| `insonmnia/cgroups/cgroup_linux.go` | Linux‑specific implementation of a cgroup, including path handling, statistics gathering and YAML configuration support. |
| `insonmnia/cgroups/cgroup_nonlinux.go` | Stub for non‑Linux platforms; currently returns a dummy `nilCgroup`. |

---

## Short summary

The package exposes two main abstractions:

* **`CGroup`** – represents a single control group, providing methods to add processes, read statistics and create child groups.
* **`CGroupManager`** – manages a collection of nested cgroups under a parent group.

On Linux the concrete type `cgroup` embeds `github.com/containerd/cgroups.Cgroup` and adds a suffix field that keeps track of the base path.  
The manager (`controlGroupManager`) holds a reference to its parent, a mutex for thread safety, and a map of child groups keyed by name.

---

## Environment variables / flags

| Variable / flag | Description |
|------------------|-------------|
| `platformSupportCGroups` (bool) | Defined in *cgroup_nonlinux.go*; indicates whether the platform supports real cgroups.  The public constructor `NewCgroupManager(name, res)` uses this flag to decide between a nil implementation and the Linux‑specific manager. |
| YAML tag `"!!map"` | Used by `SetYAML` in *cgroup_linux.go* to decode configuration maps into a `specs.LinuxResources` struct via mapstructure. |

---

## Command‑line arguments

The package itself does not expose any CLI flags, but the public constructor is:

```go
func NewCgroupManager(name string, res *specs.LinuxResources) (CGroup, CGroupManager, error)
```

* `name` – base name of the parent group.  
* `res` – optional Linux resource limits to apply when creating the group.

---

## Edge cases for launching

1. **Linux build** – The file *cgroup_linux.go* is compiled only on Linux (`// +build linux`).  It creates a real cgroup via `initializeControlGroup(name, res)` and wraps it in a `controlGroupManager`.  
2. **Non‑Linux build** – The fallback implementation in *cgroup_nonlinux.go* returns a dummy `nilCgroup`; this is useful for testing or on platforms that do not support containerd cgroups yet.  
3. **Configuration via YAML** – If the application loads configuration from a YAML file, the tag `"!!map"` will be matched by `SetYAML`, allowing you to specify Linux resources in the same map that feeds into `NewCgroupManager`.  

---

## Relations between code entities

| Entity | Relationship |
|--------|--------------|
| `cgroup` (struct) | Embeds `containerd.Cgroup`; adds a `suffix` field.  Implements the `CGroup` interface. |
| `nilCgroup` | Stub implementation of `CGroup`; used when platform support is false. |
| `controlGroupManager` | Holds a parent `CGroup`, a mutex, and a map of child groups; implements `CGroupManager`. |
| `NewCgroupManager(name, res)` | Public entry point that chooses between the nil or Linux implementation based on `platformSupportCGroups`. |
| `initializeControlGroup(name, res)` (Linux) | Low‑level helper that creates a new cgroup at path `filepath.Join(c.suffix, name)` and returns it as a `CGroup`. |
| `SetYAML(tag, value)` | Implements the `goyaml.Setter` interface; decodes YAML maps into `specs.LinuxResources` for use by `NewCgroupManager`. |

---

## File structure

```
insonmnia/cgroups/
├── cgroup.go          # core interfaces & manager logic
├── cgroup_linux.go    # Linux implementation of CGroup and helper functions
└── cgroup_nonlinux.go # Non‑Linux fallback (nil implementation)
```

---

**End of output**