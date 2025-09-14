# Cedana Checkpoint/Restore Tutorial

This collection helps you understand how checkpoint/restore works by exploring the Cedana project. You'll learn the concepts step-by-step using real code examples and practical exercises that you can run yourself.

## Tutorial Categories

### **[Basics](./basics/)** - Foundation Level
Start here if you're new to checkpoint/restore. These tutorials cover the essential concepts that everyone should understand before moving to specialized topics.
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

**Prerequisites: Basics + one specialization**

## Learning Paths

Different people need different things from checkpoint/restore technology. Pick the path that matches your role:

### **Developer/Researcher**
`basics/` → `gpu-snapshots/01-cuda-basics.md`
*Just want to understand the core concepts and maybe try GPU checkpointing*

### **AI/ML Engineer**
`basics/` → `gpu-snapshots/` → `live-migration/01-migration-strategies.md`
*Focus on moving GPU workloads and training jobs between machines*

### **DevOps Engineer**
`basics/` → `container-snapshots/` → `live-migration/`
*Need to migrate containers and deploy checkpoint/restore in production*

### **Platform Engineer**
All categories in sequence
*Want to master everything and build comprehensive migration systems*

## Lab Environment Setup

You don't need much to get started. Here's what you'll need for different tutorial paths:

### **Minimal Setup** (Basics + GPU basics)
Just need a Linux machine with Ubuntu 20.04+, 8GB RAM, and 20GB disk space. Root access required since CRIU needs low-level system access.

### **GPU Setup** (GPU snapshots)
Need an NVIDIA GPU (GTX 1060 or newer works fine) with drivers 570+ installed. You'll also need the CUDA toolkit, but the tutorials will guide you through this.

### **Container Setup** (Container snapshots)
Docker needs to be installed and running. For Kubernetes tutorials, you can use minikube or kind for learning - you don't need a full production cluster.

### **Multi-Host Setup** (Live migration)
For testing real migration scenarios, you'll want at least 2 machines that can talk to each other over the network. VMs work perfectly for this.

## Why These Tutorials Work

### **You Actually Build Things**
Instead of just reading theory, you'll install software, run commands, and see checkpoint/restore working on your own machine. Every tutorial includes complete setup steps and working examples.

### **One Thing at a Time**
Each tutorial focuses on a single concept. You won't get overwhelmed trying to learn everything at once. Most tutorials take about 15-30 minutes to complete.

### **Based on Real Code**
These aren't made-up examples. You'll explore the actual Cedana codebase (`internal/cedana/criu/`, `pkg/criu/`) and understand how a real production system implements checkpoint/restore.

## Quick Start

### **New to CRIU**
Start with [Environment Setup and Validation](./basics/01-setup-and-validation.md)

### **GPU Workload Focus**
Complete basics, then [CUDA Fundamentals](./gpu-snapshots/01-cuda-basics.md)

### **Container Migration Focus**
Complete basics, then [Docker Container Fundamentals](./container-snapshots/01-docker-basics.md)

### **Production Deployment Focus**
Complete basics + specialization, then [Live Migration](./live-migration/)

## What You'll Know After This

By the end of these tutorials, you'll understand how checkpoint/restore actually works and be able to:
- Set up CRIU on different systems and troubleshoot common issues
- Checkpoint and restore regular processes, containers, and GPU workloads
- Implement live migration strategies that minimize downtime
- Debug problems when things don't work as expected
- Deploy checkpoint/restore solutions in production environments

**Ready to dive in?** Start with [Environment Setup and Validation](./basics/01-setup-and-validation.md) to get your system ready.