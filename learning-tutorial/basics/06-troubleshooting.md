# CRIU Troubleshooting Guide

## What You'll Learn
Diagnose and fix common checkpoint/restore failures with step-by-step debugging.

## Common Issues and Solutions

### Issue 1: Permission Denied
```bash
# Problem: CRIU fails with "Operation not permitted"

# Check current permissions
sudo criu check

# Solution: Disable YAMA ptrace restrictions
sudo sysctl -w kernel.yama.ptrace_scope=0

# Make permanent
echo 'kernel.yama.ptrace_scope=0' | sudo tee -a /etc/sysctl.conf

# Test fix
echo "Testing permission fix..."
sleep 100 &
TEST_PID=$!
sudo criu dump -t $TEST_PID -D ./permission-test -v4
kill $TEST_PID
rm -rf ./permission-test
echo "✅ Permission issue fixed"
```

### Issue 2: Missing Kernel Features
```bash
# Problem: "Kernel doesn't support feature X"

# Check what's missing
sudo criu check --feature mem_dirty_track
sudo criu check --feature aio_remap

# Check kernel config
grep CONFIG_CHECKPOINT_RESTORE /boot/config-$(uname -r) || echo "❌ Feature missing"

# If missing, you need newer kernel or recompile
echo "Current kernel: $(uname -r)"
echo "Required: Linux 3.11+ with CONFIG_CHECKPOINT_RESTORE=y"
```

### Issue 3: Checkpoint Too Large
```bash
# Problem: Checkpoint takes too long or uses too much space

# Test with memory-heavy process
python3 -c "
data = bytearray(1024 * 1024 * 500)  # 500MB
print('Allocated 500MB memory')
import time
time.sleep(1000)
" &
HEAVY_PID=$!

# Time the checkpoint
echo "Timing large checkpoint..."
time sudo criu dump -t $HEAVY_PID -D ./large-checkpoint -v4

# Solution: Use pre-dump for large processes
mkdir -p ./predump-test
sudo criu pre-dump -t $HEAVY_PID -D ./predump-test
sudo criu dump -t $HEAVY_PID -D ./large-checkpoint --prev-images-dir ./predump-test

kill $HEAVY_PID 2>/dev/null
rm -rf ./large-checkpoint ./predump-test
```

### Issue 4: Process Not Restoring
```bash
# Problem: Process checkpoint created but restore fails

# Create test process with specific behavior
bash -c 'trap "echo Caught signal" SIGTERM; while true; do echo alive; sleep 1; done' &
TEST_PID=$!

# Checkpoint it
sudo criu dump -t $TEST_PID -D ./restore-test -v4

# Try restore with verbose logging
echo "Attempting restore with detailed logging..."
sudo criu restore -D ./restore-test -v4 --log-file restore.log

# Check logs for errors
if [ -f restore.log ]; then
    echo "=== Restore Log Errors ==="
    grep -i error restore.log || echo "No errors in log"
    echo "=== Log File Location ==="
    echo "Full logs: ./restore-test/restore.log"
fi

# Cleanup
pkill -f "echo alive" 2>/dev/null
rm -rf ./restore-test restore.log
```

### Issue 5: Network Socket Issues
```bash
# Problem: Process with network connections fails to checkpoint

# Create process with network socket
python3 -c "
import socket
import time
s = socket.socket()
s.bind(('127.0.0.1', 8888))
s.listen(1)
print('Server listening on port 8888')
while True:
    time.sleep(1)
" &
SOCKET_PID=$!

# This will likely fail
echo "Testing socket checkpoint (may fail)..."
sudo criu dump -t $SOCKET_PID -D ./socket-test -v4

# Solution: Close sockets before checkpoint or use --tcp-established
echo "Retrying with TCP options..."
sudo criu dump -t $SOCKET_PID -D ./socket-test --tcp-established -v4

kill $SOCKET_PID 2>/dev/null
rm -rf ./socket-test
```

## Debugging Workflow

### Step 1: Check Basic System Health
```bash
#!/bin/bash
echo "=== CRIU System Health Check ==="

# Basic functionality
echo "1. Checking CRIU installation..."
criu --version || echo "❌ CRIU not installed"

# System support
echo "2. Checking system compatibility..."
sudo criu check || echo "❌ System compatibility issues"

# Permissions
echo "3. Checking permissions..."
if [ "$EUID" -eq 0 ]; then
    echo "✅ Running as root"
else
    echo "⚠️ Not running as root - may need sudo"
fi

# Kernel version
echo "4. Checking kernel version..."
KERNEL_VER=$(uname -r | cut -d. -f1-2)
echo "Kernel: $KERNEL_VER (need 3.11+)"
```

### Step 2: Test with Simple Process
```bash
#!/bin/bash
echo "=== Simple Process Test ==="

# Start minimal test process
sleep 30 &
SIMPLE_PID=$!
echo "Started test process: $SIMPLE_PID"

# Attempt checkpoint
if sudo criu dump -t $SIMPLE_PID -D ./simple-test -v4; then
    echo "✅ Simple checkpoint successful"

    # Test restore
    if sudo criu restore -D ./simple-test -v4; then
        echo "✅ Simple restore successful"
        echo "System is working correctly"
    else
        echo "❌ Restore failed - check restore.log"
    fi
else
    echo "❌ Checkpoint failed - system has issues"
fi

# Cleanup
kill $SIMPLE_PID 2>/dev/null
rm -rf ./simple-test
```

### Step 3: Analyze Logs
```bash
#!/bin/bash
echo "=== Log Analysis Helper ==="

LOG_FILE="./test-checkpoint/criu.log"

if [ -f "$LOG_FILE" ]; then
    echo "Found log file: $LOG_FILE"

    echo "=== Common Error Patterns ==="
    echo "Permission errors:"
    grep -i "operation not permitted\|permission denied" "$LOG_FILE" || echo "None found"

    echo "Missing features:"
    grep -i "feature.*not supported\|kernel.*support" "$LOG_FILE" || echo "None found"

    echo "Memory issues:"
    grep -i "memory\|allocation\|oom" "$LOG_FILE" || echo "None found"

    echo "Network issues:"
    grep -i "socket\|connection\|network" "$LOG_FILE" || echo "None found"

else
    echo "No log file found at $LOG_FILE"
    echo "Run checkpoint with -v4 --log-file criu.log for detailed logs"
fi
```

## Quick Fix Script
```bash
#!/bin/bash
echo "=== CRIU Quick Fix Script ==="

# Fix common issues automatically
echo "Applying common fixes..."

# 1. Fix YAMA ptrace
sudo sysctl -w kernel.yama.ptrace_scope=0

# 2. Install missing packages
sudo apt update
sudo apt install -y criu build-essential

# 3. Check and report system status
echo "System status after fixes:"
sudo criu check

echo "Quick fixes applied. Test with a simple checkpoint now."
```

## When to Get Help
- Kernel missing required features → Upgrade kernel or compile with options
- Hardware-specific issues → Check CRIU documentation for your hardware
- Complex application failures → Use CRIU mailing list or forums
- Performance issues → Profile with different options and system resources

Remember: Start simple, add complexity gradually, and always check logs!