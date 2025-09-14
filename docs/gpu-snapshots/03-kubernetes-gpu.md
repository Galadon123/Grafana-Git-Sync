# Kubernetes GPU Pod Management

## What We'll Learn
How Cedana checkpoints Kubernetes pods that use GPUs, handling GPU resource scheduling and cluster coordination.

## Core Concepts

GPU pods in Kubernetes involve additional complexity:
- **GPU Resource Scheduling** - Kubernetes must allocate GPU resources to pods
- **Device Plugin Integration** - NVIDIA device plugin manages GPU availability
- **Node Affinity** - Pods must be scheduled on GPU-enabled nodes
- **Resource Quotas** - GPU allocation limits and sharing policies

## Kubernetes GPU Architecture

### NVIDIA Device Plugin
- **Resource Discovery** - Identifies available GPUs on cluster nodes
- **Resource Allocation** - Allocates GPU resources to requesting pods
- **Health Monitoring** - Monitors GPU health and availability
- **Version Compatibility** - Manages driver and plugin version compatibility

### GPU Resource Management
- **Resource Requests** - Pods request GPU resources via `nvidia.com/gpu`
- **Node Selection** - Scheduler places pods on nodes with available GPUs
- **Resource Limits** - Enforces GPU allocation limits per pod
- **Sharing Policies** - Manages GPU sharing between multiple pods

## Checkpoint Coordination

When checkpointing GPU pods, Cedana must:
1. **Validate GPU Resources** - Ensure pod actually has GPU access
2. **Coordinate with Scheduler** - Work with Kubernetes scheduler for restore placement
3. **Preserve Device Mappings** - Maintain GPU device assignments
4. **Handle Multi-Container** - Coordinate GPU access across pod containers

## Quick Implementation

1. **Deploy GPU Pod** - Create pod with GPU resource requests
2. **Verify GPU Access** - Confirm pod containers can use GPUs
3. **Checkpoint Pod** - `cedana dump --type k8s --pod <gpu-pod> --cuda`
4. **Restore Coordination** - Ensure target cluster has GPU nodes available
5. **Validate Restore** - Confirm restored pod has GPU access

## GPU Resource Configuration

### Pod Specification
```yaml
resources:
  limits:
    nvidia.com/gpu: 1
  requests:
    nvidia.com/gpu: 1
nodeSelector:
  accelerator: nvidia
```

### Multi-GPU Pods
```yaml
resources:
  limits:
    nvidia.com/gpu: 2
```

## Cluster Requirements

### NVIDIA Device Plugin
- Device plugin daemonset deployed cluster-wide
- Compatible with cluster Kubernetes version
- Proper GPU node labeling and taints

### GPU Node Configuration
- NVIDIA drivers installed on GPU nodes
- Container runtime with GPU support
- Proper device permissions and access

## Common Challenges

### Resource Allocation
- **GPU Availability** - Sufficient GPUs available for restore
- **Node Scheduling** - GPU nodes available in target cluster
- **Resource Conflicts** - Other pods using GPU resources
- **Quota Limits** - Namespace GPU resource quotas

### Cross-Cluster Migration
- **Device Plugin Compatibility** - Different plugin versions between clusters
- **Driver Versions** - GPU driver compatibility across clusters
- **Node Configurations** - Different GPU hardware configurations
- **Resource Policies** - Different GPU allocation policies

## Use Cases

### AI/ML Training at Scale
- **Distributed Training** - Multi-GPU training jobs across cluster
- **Job Migration** - Move training jobs between clusters
- **Resource Optimization** - Balance GPU workloads across nodes
- **Fault Tolerance** - Recover training jobs from GPU node failures

### GPU Service Deployment
- **Inference Services** - GPU-accelerated inference pods
- **Auto-scaling** - Scale GPU services based on demand
- **Rolling Updates** - Update GPU services with minimal downtime
- **Multi-tenancy** - Share GPU resources between different teams

## Key Files to Explore
- `plugins/k8s/` - Kubernetes GPU pod handling
- `internal/cedana/criu/checks_cuda.go` - GPU validation in cluster context
- NVIDIA device plugin configuration and management

GPU pod checkpointing enables sophisticated cluster management and workload mobility for GPU-accelerated applications.