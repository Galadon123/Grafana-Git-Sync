# Cedana CUDA Checkpoint Fundamentals

## What You'll Learn
How Cedana implements GPU workload checkpointing using NVIDIA CUDA integration, based on the project's CUDA plugin architecture.

## Cedana's CUDA Implementation

### Core CUDA Support
Cedana's CUDA integration lives in:
- `internal/cedana/criu/checks_cuda.go` - Driver and version validation
- CRIU wrapper with CUDA extensions in `pkg/criu/`
- Specialized checkpoint handlers for GPU memory and contexts

### CUDA Plugin Architecture
The Cedana CUDA plugin validates and manages:
- **Driver Version Checking**: Ensures NVIDIA driver >= 570 via `nvidia-smi` parsing
- **CRIU Version Validation**: Requires CRIU 4.0+ with CUDA support enabled
- **GPU Resource Management**: Device enumeration and memory allocation tracking
- **Context Preservation**: CUDA execution context state maintenance

## Key Concepts

### GPU State Components
When Cedana checkpoints CUDA workloads, it preserves:
- **Device Memory**: All GPU memory allocations and their contents
- **CUDA Contexts**: GPU execution environments and their configurations
- **Stream State**: Asynchronous operation queues and dependencies
- **Device Properties**: Hardware capabilities and current settings

### Version Requirements (from checks_cuda.go)
Cedana enforces strict version compatibility:
- `CRIU_MIN_VERSION_FOR_CUDA = 40000` (CRIU 4.0+)
- `CRIU_MIN_CUDA_VERSION = 570` (NVIDIA driver 570+)
- CUDA toolkit compatibility for target applications

### Checkpoint Process Flow
1. **Pre-Checkpoint Validation**: GPU driver and CRIU version checks
2. **GPU Memory Enumeration**: Scan all device memory allocations
3. **Context State Capture**: Save CUDA execution environments
4. **Memory Contents Dump**: Transfer GPU memory to checkpoint images
5. **Device Configuration Save**: Record GPU hardware settings

## Understanding Cedana's Implementation

### Driver Version Validation
Cedana parses `nvidia-smi` output to extract driver versions:
- Uses regex pattern matching to find driver version strings
- Validates against minimum required versions
- Reports specific version incompatibilities for troubleshooting

### CUDA Context Management
The checkpoint process handles:
- **Multi-Context Applications**: Applications using multiple GPU contexts
- **Context Dependencies**: Relationships between different contexts
- **Resource Allocation**: GPU memory and compute resource tracking
- **State Consistency**: Ensuring coherent GPU state during checkpoint

### Cross-GPU Migration Considerations
Cedana supports migration between:
- **Identical GPUs**: Same model and architecture (fully supported)
- **Compatible GPUs**: Same architecture, different models (limited support)
- **Different Architectures**: Requires careful compatibility validation

## Lab Exercise

### Exploring CUDA Integration
1. **Review Health Checks**: Examine how Cedana validates CUDA support
2. **Simple GPU Application**: Test with basic CUDA memory allocation
3. **Checkpoint Creation**: Use Cedana to checkpoint GPU workload
4. **Cross-System Migration**: Test restore on different GPU systems
5. **Validation**: Verify GPU memory and context preservation

### Understanding Limitations
- NVIDIA-specific implementation (no AMD/Intel GPU support yet)
- Driver version sensitivity
- GPU architecture compatibility constraints
- Memory allocation size impacts on checkpoint speed

## Success Criteria
- Understand Cedana's CUDA validation process
- Can checkpoint simple GPU applications
- Recognize version compatibility requirements
- Know when cross-GPU migration will succeed

## Reference Files
- `internal/cedana/criu/checks_cuda.go` - CUDA validation implementation
- `pkg/criu/criu.go` - Core CRIU integration with GPU support
- Plugin architecture examples for GPU-specific handling

This foundation enables you to work with Cedana's GPU checkpoint capabilities and understand its design decisions for CUDA workload migration.