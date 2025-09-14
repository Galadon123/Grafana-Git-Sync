# CRIU Restore Tutorial

## Overview
CRIU restore operation brings a checkpointed process back to life from its saved state. This technique reconstructs the entire process environment, including memory, file descriptors, and process relationships.

## How CRIU Restore Works

### Key Steps
1. **Image Reading**: Load checkpoint data from image files
2. **Memory Restoration**: Recreate process memory mappings and content
3. **File Descriptor Recovery**: Reopen files, sockets, and pipes
4. **Process Creation**: Fork and configure the restored process
5. **State Application**: Restore registers and execution context
6. **Namespace Setup**: Recreate process namespaces

### Implementation Flow (from restore_handlers.go:29)
```go
func restore(ctx context.Context, opts types.Opts, resp *daemon.RestoreResp, req *daemon.RestoreReq) {
    // Configure restore options
    criuOpts.ImagesDir = proto.String(imageDir)
    criuOpts.LogFile = proto.String("criu-restore.log")

    // Handle I/O streams for restored process
    var stdin io.Reader
    var stdout, stderr io.Writer

    // Execute restore operation
    criuResp, err := opts.CRIU.Restore(ctx, criuOpts, opts.CRIUCallback,
                                       stdin, stdout, stderr, extraFiles...)

    // Set restored process PID
    resp.PID = uint32(*criuResp.Pid)
}
```

### Process Lifecycle Management
The restore handler manages the complete lifecycle:
- **Process Reaping**: Monitors restored process exit
- **Stream Handling**: Manages stdin/stdout/stderr
- **Exit Code Tracking**: Captures process exit status

### Key Configuration Options
- `ImagesDir`: Directory containing checkpoint images
- `LogLevel`: Restore operation verbosity
- `GhostLimit`: Maximum size for ghost files
- `Attachable`: Whether process can be attached to terminal

### I/O Stream Management
Restore supports different I/O modes:
- **Daemonless**: Direct connection to terminal (stdin/stdout/stderr)
- **Attachable**: Stream I/O through cedana's I/O system
- **File Output**: Redirect output to specified file

## Restore Process Architecture

### Memory Restoration
1. Parse memory mappings from `mm-*.img`
2. Allocate virtual memory regions
3. Load page contents from `pages-*.img`
4. Set memory permissions and flags

### File Descriptor Recovery
1. Read file information from `files.img`
2. Reopen files in same order
3. Restore file positions and flags
4. Handle special files (sockets, pipes, devices)

### Namespace Recreation
1. Create new namespaces as needed
2. Configure network interfaces
3. Set up mount points
4. Restore PID namespace hierarchy

## Error Handling and Recovery

### Common Restore Failures
- **Missing files**: Files that existed during dump are now absent
- **Permission changes**: File permissions modified since dump
- **Network conflicts**: Port/address already in use
- **Resource limits**: System limits prevent restoration

### Zombie Process Cleanup
```go
if pid := resp.GetState().GetPID(); pid != 0 {
    if p, err := os.FindProcess(int(pid)); err == nil {
        p.Wait() // Clean up zombie process
    }
}
```

## Advanced Features

### Notification System
CRIU supports callback notifications during restore:
- `pre-restore`: Before starting restore
- `post-restore`: After successful restore
- `setup-namespaces`: During namespace creation
- `network-lock/unlock`: Network configuration phases

### Multi-Process Restoration
For process trees, CRIU coordinates:
- Parent-child relationship recreation
- Shared memory restoration
- Signal delivery synchronization

## When to Use CRIU Restore
- **Service migration**: Move services between servers
- **Disaster recovery**: Restore from checkpoint after failure
- **Development workflow**: Quickly return to specific application state
- **Container resumption**: Restart containers from saved state
- **Testing**: Restore consistent test environments

## Best Practices
1. **Verify image integrity** before restore
2. **Check resource availability** (memory, disk space, ports)
3. **Handle external dependencies** (databases, APIs) gracefully
4. **Monitor restored process health** after startup
5. **Implement retry logic** for transient failures
6. **Use proper logging** to debug restore issues

## Troubleshooting
- Check CRIU restore logs in `criu-restore.log`
- Verify all checkpoint image files are present
- Ensure sufficient system resources
- Test restore in isolated environment first
- Monitor process behavior after restoration