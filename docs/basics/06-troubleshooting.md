# Troubleshooting Guide

## What We'll Learn
How to diagnose and fix common checkpoint/restore issues using systematic debugging approaches.

## Common Issues and Solutions

### Permission Problems
- **Symptom** - "Operation not permitted" errors
- **Cause** - YAMA ptrace restrictions or insufficient privileges
- **Fix** - Run as root or adjust `kernel.yama.ptrace_scope=0`

### Missing Kernel Features
- **Symptom** - "Kernel doesn't support feature X"
- **Cause** - Linux kernel without CONFIG_CHECKPOINT_RESTORE
- **Fix** - Use kernel 3.11+ with checkpoint/restore support enabled

### Large Checkpoint Issues
- **Symptom** - Checkpoint takes too long or fails
- **Cause** - Process has massive memory footprint
- **Fix** - Use pre-dump techniques or increase system resources

### Network Socket Failures
- **Symptom** - Checkpoint fails on processes with network connections
- **Cause** - CRIU cannot handle some network socket types
- **Fix** - Close sockets before checkpoint or use --tcp-established flag

## Debugging Workflow

### Step 1: Health Check
Run `cedana health-check` to identify system-level issues before attempting checkpoints.

### Step 2: Start Simple
Test with basic processes (like `sleep`) before trying complex applications.

### Step 3: Check Logs
Examine CRIU logs with verbose output (`-v4`) to understand failure points.

### Step 4: Isolate Variables
- Test on same machine first
- Try without network connections
- Use minimal test cases

## Common Log Patterns

- **"Permission denied"** - Check user privileges and YAMA settings
- **"Feature not supported"** - Verify kernel configuration
- **"Memory allocation failed"** - Insufficient system memory
- **"Network socket"** - Application has active network connections

## Quick Fixes

1. **Basic Setup** - Ensure CRIU installed and kernel supports checkpoint/restore
2. **Permissions** - Run as root or configure capabilities properly
3. **Simple Test** - Start with `sleep` process to verify basic functionality
4. **Log Analysis** - Use verbose logging to understand specific failures

## Key Files to Explore
- `internal/cedana/criu/checks.go` - System validation logic
- CRIU log files provide detailed error information

Systematic debugging helps identify whether issues are environmental, application-specific, or configuration-related.