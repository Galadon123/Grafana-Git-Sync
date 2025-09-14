# Docker Container Fundamentals

## What We'll Learn
How Cedana checkpoints entire Docker containers using its containerd plugin architecture.

## Core Concepts

Container checkpointing is more complex than process checkpointing because containers include:
- **Process Tree** - All processes running inside the container
- **Filesystem State** - Container layer changes from the base image
- **Network Configuration** - Virtual interfaces, IP addresses, routing
- **Resource Limits** - Memory, CPU, and I/O constraints
- **Security Context** - User namespaces, capabilities, SELinux labels

## How Container Checkpointing Works

Cedana's containerd plugin:
1. **Connects** to containerd daemon via API
2. **Sets Namespace** - Ensures we're working in the correct containerd namespace
3. **Captures State** - Uses CRIU to dump all container processes
4. **Preserves Metadata** - Saves container configuration and runtime state
5. **Coordinates Images** - Ensures base images are available for restore

## Plugin Architecture

Cedana uses a plugin system for different container runtimes:
- **Containerd Plugin** - Direct integration with containerd daemon
- **Kubernetes Plugin** - Pod-level operations in K8s clusters
- **CRI-O Plugin** - Support for CRI-O container runtime

## When to Use Container Checkpointing

- **Container Migration** - Move containers between hosts
- **Development Workflows** - Save development environment states
- **Testing** - Create reproducible test environments
- **Disaster Recovery** - Backup container states for recovery

## Quick Implementation

1. **Find Container** - `docker ps` to identify running container
2. **Get Container ID** - Note the container ID from containerd namespace
3. **Checkpoint** - `cedana dump --type containerd --namespace moby --id <container-id>`
4. **Restore** - `cedana restore --type containerd --checkpoint-dir <path>`

## Container-Specific Challenges

- **Image Dependencies** - Base images must exist on target system
- **Volume Management** - Persistent and temporary volume handling
- **Network Reconfiguration** - Port conflicts and IP address changes
- **State vs Stateless** - Different approaches for different container types

## Key Files to Explore
- `plugins/containerd/internal/client/dump_adapters.go` - Container setup logic
- `plugins/containerd/cmd/dump.go` - CLI container operations
- `plugins/k8s/` - Kubernetes-specific container handling

Container checkpointing enables seamless migration of entire application environments.