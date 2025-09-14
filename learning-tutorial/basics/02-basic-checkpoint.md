# CRIU Dump/Checkpoint Tutorial

## Overview
CRIU (Checkpoint/Restore in Userspace) dump operation creates a snapshot of a running process by saving its entire state to disk. This technique allows you to pause a process and resume it later, even on a different machine.

## How CRIU Dump Works

### Key Components
- **Process Memory**: All memory pages used by the process
- **File Descriptors**: Open files, sockets, pipes
- **Process Tree**: Parent-child relationships
- **Namespaces**: Network, mount, PID, and other namespaces
- **Credentials**: User/group IDs and capabilities

### Implementation Flow
1. **Process Discovery**: Find target process by PID
2. **Memory Dumping**: Save all memory mappings and pages
3. **File Descriptor Collection**: Capture all open file handles
4. **State Serialization**: Save process registers and context
5. **Image Creation**: Write checkpoint data to image files

### Code Example (from dump_handlers.go:37)
```go
func dump(ctx context.Context, opts types.Opts, resp *daemon.DumpResp, req *daemon.DumpReq) {
    // Set CRIU options
    criuOpts.Pid = proto.Int32(int32(resp.GetState().GetPID()))
    criuOpts.ImagesDir = proto.String(dumpDir)
    criuOpts.LogFile = proto.String("criu-dump.log")

    // Execute dump operation
    _, err = opts.CRIU.Dump(ctx, criuOpts, opts.CRIUCallback)
}
```

### Key Configuration Options
- `ImagesDir`: Directory where checkpoint images are stored
- `Pid`: Target process ID to dump
- `GhostLimit`: Maximum size for "ghost" files (200MB default)
- `LogLevel`: Verbosity of CRIU logging

### Output Files
After a successful dump, you'll find these files in the images directory:
- `core-*.img`: Process core information
- `mm-*.img`: Memory mappings
- `pages-*.img`: Memory page contents
- `files.img`: File descriptor information
- `criu-dump.log`: Operation log

## When to Use CRIU Dump
- **Long-running computations**: Save progress and resume later
- **Live migration**: Move processes between servers
- **Container checkpointing**: Save container state
- **Debugging**: Capture process state for analysis
- **Fault tolerance**: Create recovery points

## Limitations
- Requires root privileges or specific capabilities
- Some system calls cannot be checkpointed
- External dependencies (databases, network connections) may break
- Large memory processes create large checkpoint files

## Best Practices
1. Test checkpoint/restore in development environment first
2. Monitor disk space for checkpoint images
3. Handle external resource reconnection after restore
4. Use pre-dump for faster final dumps on large processes