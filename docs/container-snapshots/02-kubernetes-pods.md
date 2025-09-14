# Kubernetes Pod Management

## What We'll Learn
How Cedana checkpoints entire Kubernetes pods, including multi-container pods, persistent volumes, and cluster coordination.

## Core Concepts

Pod checkpointing is more complex than single containers because pods include:
- **Multi-Container Architecture** - Main containers, sidecars, and init containers
- **Shared Resources** - Network namespace, storage volumes, and IPC
- **Cluster Integration** - Service discovery, load balancing, and scheduling
- **Persistent Volumes** - Stateful data that survives pod restarts

## How Pod Checkpointing Works

Cedana's Kubernetes plugin:
1. **Identifies Pod Components** - Enumerates all containers within the pod
2. **Coordinates Checkpoint** - Handles container dependencies and order
3. **Preserves Cluster State** - Saves network configuration and service relationships
4. **Manages Storage** - Handles persistent volume claims and data
5. **Updates Scheduler** - Coordinates with Kubernetes scheduler for restore placement

## Pod-Specific Challenges

### Multi-Container Coordination
- **Dependency Order** - Some containers depend on others being ready
- **Shared Volumes** - Multiple containers may share the same storage
- **Network Sharing** - All containers share the pod's network namespace

### Cluster Integration
- **Service Discovery** - Pod IP addresses may change after restore
- **Load Balancer Updates** - Traffic routing needs to be updated
- **Node Placement** - Scheduler must find appropriate nodes for restore

## Quick Implementation

1. **Find Pod** - `kubectl get pods` to identify target pod
2. **Checkpoint Pod** - `cedana dump --type k8s --namespace default --pod webapp-123`
3. **Transfer State** - Move checkpoint files to target cluster
4. **Restore Pod** - `cedana restore --type k8s --checkpoint-dir ./pod-checkpoint`

## StatefulSet Considerations

For stateful applications:
- **Ordered Deployment** - Pods must start in specific order
- **Persistent Volume Claims** - Storage must be available on target
- **Headless Services** - Direct pod-to-pod communication patterns

## When to Use Pod Checkpointing

- **Cluster Migration** - Move workloads between Kubernetes clusters
- **Node Maintenance** - Migrate pods before node updates
- **Development/Testing** - Create consistent test environments
- **Disaster Recovery** - Backup and restore application states

## Key Files to Explore
- `plugins/k8s/` - Kubernetes-specific plugin implementation
- `plugins/k8s/pkg/kube/container.go` - Pod container management
- Kubernetes API integration for cluster coordination

Pod checkpointing enables sophisticated workload migration within and between Kubernetes clusters.