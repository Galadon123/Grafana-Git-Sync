# GPU Container Checkpointing

## What We'll Learn
How Cedana combines GPU checkpointing with container technology to migrate Docker containers that use NVIDIA GPUs.

## Core Concepts

GPU containers require both container and GPU state preservation:
- **Container State** - Standard container processes, filesystems, and networking
- **GPU Device Access** - Container's access to GPU devices through runtime
- **CUDA Context** - GPU execution contexts within the containerized application
- **GPU Memory** - Device memory allocations made by the containerized process

## NVIDIA Container Runtime Integration

GPU containers use specialized runtime:
- **NVIDIA Container Runtime** - Docker integration that provides GPU access
- **Device Mapping** - Maps host GPU devices into container namespace
- **Driver Coordination** - Ensures container sees correct NVIDIA drivers
- **Resource Limits** - GPU allocation and sharing between containers

## Checkpoint Coordination

When checkpointing GPU containers, Cedana must:
1. **Coordinate Runtimes** - Work with both containerd and NVIDIA runtime
2. **Preserve GPU Access** - Maintain container's GPU device permissions
3. **Capture Dual State** - Both container state and GPU state simultaneously
4. **Handle Dependencies** - Manage GPU driver and CUDA library dependencies

## Quick Implementation

1. **Setup GPU Runtime** - Configure Docker with NVIDIA container runtime
2. **Run GPU Container** - Start container with `--gpus all` flag
3. **Verify GPU Access** - Confirm container can access GPU devices
4. **Checkpoint** - `cedana dump --type containerd --id <container> --cuda`
5. **Restore** - Ensure target system has compatible GPU setup

## Common Challenges

### Runtime Configuration
- **Docker Daemon** - Must be configured with NVIDIA runtime as default
- **Container Images** - Need CUDA toolkit and libraries installed
- **Device Permissions** - Container needs proper GPU device access
- **Driver Versions** - Host and container driver compatibility

### Migration Considerations
- **Target GPU** - Destination must have compatible GPU hardware
- **Driver Compatibility** - NVIDIA driver versions must match requirements
- **Runtime Setup** - Target system needs NVIDIA container runtime configured
- **Image Availability** - Base container images must be available

## Use Cases

### AI/ML Workloads in Containers
- **Training Jobs** - Long-running ML training in containers
- **Inference Services** - GPU-accelerated inference containers
- **Jupyter Notebooks** - Data science environments with GPU access
- **Deep Learning Frameworks** - TensorFlow, PyTorch containers

### Development Workflows
- **GPU Environment Consistency** - Reproducible GPU development environments
- **Testing Scenarios** - Consistent GPU testing across different machines
- **Resource Sharing** - Move GPU workloads between different GPU hosts
- **Debug Scenarios** - Capture and analyze GPU application states

## Configuration Requirements

### Docker Configuration
- NVIDIA Container Runtime installed and configured
- Docker daemon configured to use NVIDIA runtime
- Proper GPU device permissions

### Container Requirements
- Base images with CUDA toolkit installed
- Application built with GPU support
- Proper GPU resource requests in container configuration

## Key Files to Explore
- `plugins/containerd/` - Container runtime integration
- `internal/cedana/criu/checks_cuda.go` - GPU validation within containers
- NVIDIA Container Runtime documentation for setup details

GPU container checkpointing enables sophisticated AI/ML deployment patterns with full state preservation.