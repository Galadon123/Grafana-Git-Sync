# Basic Process Checkpoint

## What We'll Learn
How Cedana captures the complete state of a running process and saves it to disk for later restoration.

## Core Concepts

A checkpoint is like taking a snapshot of a running program. Cedana uses CRIU to capture:
- **Process Memory** - All data the program has in RAM
- **CPU State** - Current instruction pointer and register values
- **Open Files** - File descriptors and their current positions
- **Network Connections** - Active sockets and their state

## How It Works

When we checkpoint a process, Cedana:
1. **Pauses** the target process temporarily
2. **Scans** its memory, files, and system state
3. **Writes** all state information to image files
4. **Resumes** the process (or terminates it for migration)

The checkpoint creates several files containing different aspects of the process state.

## Quick Implementation

1. **Find Process** - Identify the PID of our target process
2. **Run Checkpoint** - `cedana dump --type process --pid <PID>`
3. **Verify** - Check that checkpoint files were created
4. **Test** - Try restoring to confirm it worked

## Key Files to Explore
- `internal/cedana/criu/dump_handlers.go` - Main checkpoint logic
- `pkg/criu/criu.go` - CRIU wrapper functions
- `internal/cedana/criu/dump_adapters.go` - Process preparation

The checkpoint files contain everything needed to recreate the exact process state later, even on a different machine.