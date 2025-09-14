# CUDA Fundamentals

## What We'll Learn
How Cedana extends checkpoint/restore to include NVIDIA GPU state, enabling migration of CUDA applications.

## Core Concepts

GPU checkpointing is more complex than CPU-only processes because we need to preserve:
- **GPU Memory** - All device memory allocations and their contents
- **CUDA Contexts** - GPU execution environments and configurations
- **Stream State** - Asynchronous operation queues and dependencies
- **Device Settings** - Hardware configuration and current state

## How GPU Checkpointing Works

Cedana's CUDA integration:
1. **Validates** NVIDIA driver version (570+ required) and CRIU version (4.0+)
2. **Enumerates** all GPU memory allocations in the target process
3. **Captures** CUDA context state and device configurations
4. **Transfers** GPU memory contents to checkpoint files
5. **Coordinates** with CPU checkpoint for complete application state

## Version Requirements

From Cedana's implementation:
- **NVIDIA Driver**: Version 570 or higher
- **CRIU Version**: 4.0+ with CUDA support compiled in
- **Hardware**: CUDA-compatible GPU (compute capability varies)

## When GPU Checkpointing is Useful

- **AI/ML Training** - Save progress during long training runs
- **Scientific Computing** - Migrate HPC applications between GPU nodes
- **Development** - Save complex GPU application states for debugging
- **Resource Management** - Move GPU workloads to optimal hardware

## Quick Implementation

1. **Verify Support** - `cedana health-check cuda` validates environment
2. **Run GPU Application** - Start CUDA program and identify its PID
3. **Checkpoint** - `cedana dump --type process --pid <PID> --cuda`
4. **Migrate & Restore** - Move checkpoint files and restore on GPU-enabled system

## Limitations

- NVIDIA-only (no AMD or Intel GPU support currently)
- Driver version sensitivity between checkpoint and restore systems
- Some CUDA features may not be fully preserved
- Cross-architecture migration has compatibility constraints

## Key Files to Explore
- `internal/cedana/criu/checks_cuda.go` - GPU validation and version checking
- Cedana's CRIU integration handles GPU memory transfer

GPU checkpointing enables powerful new workflows for GPU-accelerated applications.