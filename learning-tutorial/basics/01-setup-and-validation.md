# Cedana Environment Setup & Validation

## What You'll Learn
How Cedana validates system compatibility and prepares environments for checkpoint/restore operations using its health check system.

## Cedana Architecture Overview

### Core Components
- **Cedana Daemon** (`internal/daemon/`) - Main service handling requests
- **CRIU Integration** (`internal/cedana/criu/`, `pkg/criu/`) - Low-level checkpoint operations
- **Plugin System** (`plugins/`) - Specialized handlers for different workload types
- **Client CLI** (`cmd/`) - User interface for checkpoint/restore operations

### Health Check System
Cedana implements comprehensive health checks in `internal/cedana/criu/checks.go` and `checks_cuda.go` to validate:
- CRIU binary availability and version compatibility
- Required kernel features and permissions
- GPU drivers and CUDA support (when applicable)
- Container runtime integration

## Key Concepts

### System Requirements Validation
Cedana checks multiple system layers:
- **Kernel Features**: Required checkpoint/restore kernel options
- **CRIU Compatibility**: Version requirements and feature support
- **Permission Setup**: Root access or specific capabilities
- **Hardware Support**: GPU drivers for CUDA workloads

### Plugin-Specific Validation
Different Cedana plugins have specific requirements:
- **Process Plugin**: Basic CRIU functionality
- **Containerd Plugin**: Container runtime integration
- **K8s Plugin**: Kubernetes cluster access
- **CUDA Plugin**: NVIDIA driver version 570+ and CRIU 4.0+

### Environment Consistency
Cedana ensures checkpoint/restore works across different environments by:
- Validating compatible software versions
- Checking resource availability
- Verifying network connectivity (for migration)
- Testing basic checkpoint operations

## Lab Exercise

### Understanding Cedana's Validation
1. **Install Cedana Daemon**: Set up the main service
2. **Run Health Checks**: Use `cedana health-check` to validate system
3. **Identify Issues**: Understand what each validation failure means
4. **Fix Environment**: Apply necessary system configurations
5. **Verify Setup**: Confirm all checks pass

### Testing Different Configurations
- Basic process checkpointing capability
- Container runtime integration
- GPU support (if available)
- Multi-host connectivity (for migration)

## Success Criteria
- All Cedana health checks pass
- Can communicate with CRIU subsystem
- Plugin-specific requirements satisfied
- Ready to proceed with actual checkpoint operations

## Reference Files
- `internal/cedana/criu/checks.go` - Core health check implementation
- `internal/cedana/criu/checks_cuda.go` - GPU-specific validation
- `cmd/health-check.go` - CLI health check command
- Plugin directories for runtime-specific checks

This foundation ensures your Cedana environment is properly configured for all subsequent checkpoint/restore operations.