# Basic Process Restore

## What We'll Learn
How Cedana brings a checkpointed process back to life from saved checkpoint files.

## Core Concepts

Restore is the opposite of checkpoint - we recreate a process from its saved state. Cedana reads the checkpoint files and:
- **Recreates Memory** - Allocates and fills memory with saved data
- **Restores CPU State** - Sets registers and instruction pointer
- **Reopens Files** - Recreates file descriptors at correct positions
- **Rebuilds Connections** - Reestablishes network sockets where possible

## How It Works

During restore, Cedana:
1. **Reads** checkpoint image files
2. **Allocates** memory and system resources
3. **Creates** new process with saved state
4. **Jumps** to the exact instruction where checkpoint happened

The process continues running as if nothing interrupted it.

## Quick Implementation

1. **Locate Checkpoint** - Find the directory with checkpoint files
2. **Run Restore** - `cedana restore --checkpoint-dir <path>`
3. **Monitor** - Watch for the restored process to start
4. **Validate** - Verify the process is running correctly

## Common Challenges

- **File Dependencies** - Original files must still exist or be accessible
- **Network State** - Some connections can't be restored and need reconnection
- **Resource Conflicts** - Ports or resources might be in use
- **Environment Differences** - System differences between checkpoint and restore

## Key Files to Explore
- `internal/cedana/criu/restore_handlers.go` - Main restore logic
- `internal/cedana/criu/restore_adapters.go` - Process recreation
- `pkg/criu/criu.go` - CRIU restore functions

Successful restore means the process picks up exactly where it left off, with all its memory and state intact.