# Cedana Environment Setup & Validation

## What We'll Learn
How Cedana validates system compatibility and prepares environments for checkpoint/restore operations using its health check system.

## Core Concepts

Cedana uses a health check system to ensure our environment is ready for checkpoint/restore operations. The main components are:

- **Cedana Daemon** - Handles checkpoint/restore requests
- **CRIU Integration** - Low-level process checkpointing
- **Plugin System** - Specialized handlers for containers, GPU workloads, etc.

## What Gets Validated

When we run `cedana health-check`, it validates:
- CRIU installation and version compatibility
- Linux kernel features for checkpoint/restore
- Proper permissions (root access or capabilities)
- Hardware support (GPU drivers if needed)

## Quick Implementation

1. **Install Cedana** - Set up the daemon service
2. **Run Health Check** - `cedana health-check` shows any issues
3. **Fix Problems** - Install missing components or fix permissions
4. **Verify** - All checks should pass before proceeding

## Key Files to Explore
- `internal/cedana/criu/checks.go` - Main validation logic
- `internal/cedana/criu/checks_cuda.go` - GPU validation
- `plugins/` directories - Plugin-specific requirements

Once our health checks pass, we're ready to start checkpointing processes and containers.