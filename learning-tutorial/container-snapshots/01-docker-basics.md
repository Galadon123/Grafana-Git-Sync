# Cedana Container Checkpoint Fundamentals

## What You'll Learn
How Cedana implements container checkpointing through its containerd plugin architecture, enabling Docker container migration and state preservation.

## Cedana's Container Integration

### Plugin Architecture
Cedana's container support is built on a plugin system located in `plugins/`:
- **Containerd Plugin** (`plugins/containerd/`) - Direct containerd integration
- **Kubernetes Plugin** (`plugins/k8s/`) - Kubernetes pod management
- **Kata Plugin** (`plugins/kata/`) - Secure container support

### Core Container Components
The containerd plugin in `plugins/containerd/internal/client/` handles:
- **Container Discovery**: Finding and identifying running containers
- **Runtime Integration**: Working with containerd daemon and runc
- **State Management**: Preserving container filesystem and process state
- **Network Coordination**: Managing container network namespaces

## Key Concepts

### Container State Preservation
When Cedana checkpoints containers, it captures:
- **Process Tree**: All processes within the container namespace
- **Filesystem Changes**: Container layer modifications from base image
- **Network Configuration**: Virtual interfaces, IP addresses, routing rules
- **Resource Limits**: Memory, CPU, and I/O constraints
- **Security Context**: User namespaces, capabilities, and SELinux labels

### Cedana's Container Workflow (from dump_adapters.go)
1. **Container Runtime Setup**: Establish connection to containerd daemon
2. **Namespace Context**: Set proper containerd namespace for operations
3. **Container State Capture**: Use CRIU to dump container processes
4. **Metadata Preservation**: Save container configuration and runtime state
5. **Image Coordination**: Ensure base images are available for restore

### Runtime Integration Patterns
Cedana integrates with container runtimes through:
- **Containerd Client**: Direct API communication with containerd daemon
- **Namespace Management**: Proper isolation between different container groups
- **Runtime Context**: Maintaining container runtime-specific configurations
- **Resource Tracking**: Monitoring and preserving resource allocations

## Understanding Cedana's Implementation

### Container Setup Process
The `SetupForDump` function in `plugins/containerd/internal/client/dump_adapters.go`:
- Creates containerd client connection
- Sets appropriate namespace context
- Prepares container for checkpoint operation
- Handles container-specific metadata

### Plugin Coordination
Cedana's plugin system allows:
- **Runtime Flexibility**: Support for different container runtimes
- **Modular Design**: Separate plugins for different use cases
- **Extensibility**: Easy addition of new container runtime support
- **Configuration Management**: Runtime-specific settings and options

### State Management Challenges
Container checkpointing involves:
- **Image Dependencies**: Ensuring base images exist on target systems
- **Volume Management**: Handling persistent and temporary volumes
- **Network Reconfiguration**: Re-establishing network connectivity
- **Permission Mapping**: Maintaining user and security contexts

## Lab Exercise

### Exploring Container Integration
1. **Plugin Architecture**: Examine Cedana's container plugin structure
2. **Containerd Integration**: Test basic container checkpoint operations
3. **State Validation**: Verify container state preservation
4. **Cross-Host Migration**: Test container migration between systems
5. **Troubleshooting**: Understand common container checkpoint issues

### Container-Specific Considerations
- **Stateless vs Stateful**: Different approaches for different container types
- **Volume Dependencies**: Understanding persistent volume implications
- **Network Dependencies**: Managing external service connections
- **Base Image Requirements**: Ensuring image availability across systems

## Success Criteria
- Understand Cedana's plugin architecture for containers
- Can checkpoint basic Docker containers using Cedana
- Recognize container-specific migration challenges
- Know how to validate container state after restore

## Reference Files
- `plugins/containerd/internal/client/dump_adapters.go` - Container setup logic
- `plugins/containerd/cmd/dump.go` - CLI container dump implementation
- `plugins/k8s/` - Kubernetes-specific container handling
- `internal/cedana/criu/dump_handlers.go` - Core checkpoint logic

This foundation enables you to work with Cedana's container capabilities and understand how it abstracts container runtime complexity for seamless migration.