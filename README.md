# Kernel Systems Projects (Linux 6.14)

### Project: Process/Thread Tree Introspection System Call

Extended Linux 6.14 with a custom system call that exports structured process and thread information for a requested subtree of the process hierarchy. The kernel implementation traverses the process tree in breadth-first order, groups threads by owning process, and emits a stable ordering that user space can reconstruct into a correct parent/child hierarchy. The interface is exposed via a clean user-kernel ABI (UAPI struct) and includes careful validation of pointers, sizes, and PIDs before copying results back to user space.

Built a companion user-space tool to invoke the syscall and present the snapshot in a human-readable form, with attention to correctness under concurrent process creation/termination and robust error handling.

### Project: Kernel Process State Tracing (Ring Buffer + Control API)

Implemented a lightweight kernel tracing facility that records process state transitions into a bounded, fixed-size ring buffer. Added a control API to enable/disable tracing (optionally for a specific PID or for all processes) and to retrieve trace records from user space for debugging and analysis. The tracing core is synchronized for correctness under concurrency: state-change events can be recorded across CPUs while user space requests consistent snapshots without corrupting trace entries.

The design emphasizes real kernel constraints—bounded memory usage, stable record ordering, and safe interaction between kernel writers and user-space readers—while handling edge cases such as toggling tracing state, racing control calls, and process exit during tracing.

### Project: eBPF Scheduler Measurement + Custom Scheduling Policy

Built an end-to-end workflow for designing and evaluating scheduling behavior on Linux. First, wrote an eBPF tracing script to capture scheduler events and compute task-level metrics such as run-queue latency and completion time, providing a reproducible baseline for the default Linux scheduler (CFS). Then implemented a custom scheduling policy/class that integrates into the kernel scheduler framework and prioritizes tasks using configurable weights supplied via sched_setscheduler.

The policy is designed to minimize average completion time on weighted workloads (notably improving over CFS on a Fibonacci task set), while remaining compatible with existing scheduling classes. Emphasis was placed on kernel stability (boot safety, correct class wiring, predictable preemption) and on measurement-driven validation using the eBPF instrumentation.

### Project: Live Memory-Mapping Inspector (Shadow Page Table via Shared Mapping)

Implemented a real-time memory inspection facility that exposes a continuously updated view of a target process’s address space to a privileged inspector process. Added syscalls to establish and tear down a shared kernel <-> userspace mapping by remapping kernel-allocated memory into user space (remap_pfn_range), enabling low-overhead inspection without repeated syscall polling. The kernel maintains a shadow representation of VMAs/page mappings and updates it on page faults and mapping events, keeping the userspace-visible view in sync with the target’s evolving memory state.

Hardened the implementation with strict argument validation, permission checks, robust cleanup on error paths, and concurrency control to enforce global invariants even under racing callers.

### Project: EZFS Filesystem Kernel Module (Modern VFS Interfaces)

Implemented a filesystem as a Linux kernel module and integrated it with the VFS using modern interfaces. The module registers a filesystem type, mounts/unmounts safely, initializes superblock/inode state correctly, and supports core operations needed for directory enumeration and file access through standard user tools. The implementation targets modern 64-bit kernels and avoids deprecated interfaces (e.g., buffer heads), instead leveraging contemporary patterns such as filesystem contexts and iomap-based integration.

Validated functionality end-to-end using disk images attached via loop devices, ensuring the module loads cleanly, mounts reliably, and behaves correctly under standard workflows (mount, ls, stat, etc.).

---

*This code was developed for a university course and can’t be made public due to academic integrity/policy constraints. I can share access privately upon request.*
