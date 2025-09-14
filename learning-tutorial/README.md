# Cedana Checkpoint/Restore Tutorial

This collection helps us understand how checkpoint/restore works by exploring the Cedana project. We'll learn the concepts step-by-step using real code examples and practical exercises that we can run ourselves.

## Tutorial Categories

### **[Basics](./basics/)** - Foundation Level
We start here if we're new to checkpoint/restore. These tutorials cover the essential concepts we should understand before moving to specialized topics.
- **[Environment Setup and Validation](./basics/01-setup-and-validation.md)** - Install and verify CRIU
- **[Basic Process Checkpoint](./basics/02-basic-checkpoint.md)** - Your first checkpoint
- **[Basic Process Restore](./basics/03-basic-restore.md)** - Your first restore
- **[Memory Optimization Techniques](./basics/04-memory-optimization.md)** - Pre-dump techniques
- **[Page Server Implementation](./basics/05-page-server.md)** - Network-based migration
- **[Troubleshooting Guide](./basics/06-troubleshooting.md)** - Debug common issues



**Prerequisites: Linux fundamentals**

### **[GPU Checkpointing](./gpu-snapshots/)** - GPU Specialization
If you work with AI, machine learning, or other GPU-accelerated applications, these tutorials show you how to checkpoint and migrate your GPU workloads without losing state.
- **[CUDA Fundamentals](./gpu-snapshots/01-cuda-basics.md)** - Basic GPU checkpoint operations
- **[GPU Container Checkpointing](./gpu-snapshots/02-docker-gpu.md)** - GPU-enabled containers
- **[Kubernetes GPU Pod Management](./gpu-snapshots/03-kubernetes-gpu.md)** - GPU pods in Kubernetes
- **[AI Training Checkpointing](./gpu-snapshots/04-ai-training-checkpoint.md)** - Machine learning workloads

**Prerequisites: Basics + NVIDIA GPU**

### **[Container Checkpointing](./container-snapshots/)** - Container Specialization
Working with Docker or Kubernetes? Learn how to checkpoint entire containers, preserve their state, and migrate them between different environments seamlessly.
- **[Docker Container Fundamentals](./container-snapshots/01-docker-basics.md)** - Basic container operations
- **[Kubernetes Pod Management](./container-snapshots/02-kubernetes-pods.md)** - Kubernetes workloads
- **[Stateful Application Handling](./container-snapshots/03-stateful-apps.md)** - Databases and caches
- **[Web Application Migration](./container-snapshots/04-web-app-migration.md)** - Live service migration

**Prerequisites: Basics + Docker/Kubernetes**

### **[Live Migration](./live-migration/)** - Production Level
Ready to deploy in production? These tutorials cover advanced scenarios like zero-downtime migration, handling complex application stacks, and implementing robust testing strategies.
- **[Migration Strategy Implementation](./live-migration/01-migration-strategies.md)** - Zero-downtime patterns
- **[Enterprise Deployment Scenarios](./live-migration/02-enterprise-scenarios.md)** - Complex deployments
- **[Validation and Testing Framework](./live-migration/03-validation-testing.md)** - Production testing
- **[Multi-Service Stack Migration](./live-migration/04-multi-service-migration.md)** - Application stacks
