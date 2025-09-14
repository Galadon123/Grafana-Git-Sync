# Tutorial 11: GPU Container Checkpointing

## Overview
Combine GPU checkpointing with container technology - checkpoint Docker containers that use NVIDIA GPUs for AI/ML workloads.

## Prerequisites
- Completed tutorials 1-3, 6, 7 (Foundation + GPU + Container basics)
- NVIDIA GPU with drivers 570+
- Docker with NVIDIA container runtime
- CUDA-enabled container images

## Key Concepts

### GPU Container Architecture
- **NVIDIA Container Runtime**: Docker integration with GPU access
- **CUDA Container Images**: Pre-built images with CUDA toolkit
- **GPU Resource Management**: Container GPU allocation and limits
- **Device Mapping**: GPU device access within containers

### Combined Checkpoint Challenges
- **GPU State + Container State**: Dual-layer checkpoint complexity
- **Device Access Preservation**: Maintaining GPU access after restore
- **Container Runtime Coordination**: Working with NVIDIA container runtime
- **Resource Scheduling**: GPU allocation during container restore

## What You'll Learn

### GPU Container Setup
- Configuring Docker for GPU access
- Running CUDA applications in containers
- Managing GPU resources and limits
- Monitoring GPU usage within containers

### Checkpoint Process
- Coordinating GPU and container checkpointing
- Handling GPU device permissions in containers
- Managing CUDA context within containerized applications
- Preserving GPU memory alongside container state

### Restoration Workflow
- Restoring containers with GPU access
- Re-establishing GPU device mappings
- Validating GPU functionality post-restore
- Troubleshooting GPU access issues

## Test Lab Setup

### Environment Preparation
```bash
# Install NVIDIA container runtime
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
sudo apt-get update && sudo apt-get install -y nvidia-container-runtime

# Configure Docker daemon
sudo vim /etc/docker/daemon.json
# Add: {"default-runtime": "nvidia", "runtimes": {"nvidia": {"path": "nvidia-container-runtime", "runtimeArgs": []}}}

# Restart Docker
sudo systemctl restart docker
```

### Test Container
```bash
# Pull CUDA test image
docker pull nvidia/cuda:11.8-runtime-ubuntu20.04

# Run GPU test container
docker run --gpus all -d --name gpu-test \
    nvidia/cuda:11.8-runtime-ubuntu20.04 \
    bash -c "while true; do nvidia-smi; sleep 10; done"

# Verify GPU access
docker logs gpu-test
```

### Checkpoint Test
```bash
# Get container ID
CONTAINER_ID=$(docker ps -q -f name=gpu-test)

# Checkpoint GPU container
cedana dump --type containerd --namespace moby --id $CONTAINER_ID --cuda

# Stop original container
docker stop gpu-test
docker rm gpu-test

# Restore GPU container
cedana restore --type containerd --checkpoint-dir ./gpu-test-checkpoint

# Verify GPU access in restored container
docker logs $(docker ps -q -l)
```

## AI/ML Workload Examples

### TensorFlow GPU Container
```bash
# Run TensorFlow training container
docker run --gpus all -d --name tf-training \
    tensorflow/tensorflow:latest-gpu-py3 \
    python -c "
import tensorflow as tf
print('GPUs:', tf.config.list_physical_devices('GPU'))
# Simple training loop for checkpoint testing
model = tf.keras.Sequential([tf.keras.layers.Dense(1)])
data = tf.random.normal([1000, 10])
labels = tf.random.normal([1000, 1])
model.compile(optimizer='adam', loss='mse')
model.fit(data, labels, epochs=100, verbose=1)
"

# Checkpoint during training
cedana dump --type containerd --id $(docker ps -q -f name=tf-training) --cuda
```

### PyTorch GPU Container
```bash
# Run PyTorch container with training
docker run --gpus all -d --name pytorch-training \
    pytorch/pytorch:latest \
    python -c "
import torch
print('CUDA available:', torch.cuda.is_available())
print('GPU count:', torch.cuda.device_count())
# Simple training for testing
x = torch.randn(1000, 20).cuda()
y = torch.randn(1000, 1).cuda()
model = torch.nn.Linear(20, 1).cuda()
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
for epoch in range(1000):
    pred = model(x)
    loss = torch.nn.functional.mse_loss(pred, y)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    if epoch % 100 == 0:
        print(f'Epoch {epoch}, Loss: {loss.item()}')
"

# Test checkpoint/restore
cedana dump --type containerd --id $(docker ps -q -f name=pytorch-training) --cuda
```

## Common Use Cases

### Development Workflows
- **Experiment Checkpointing**: Save training progress during development
- **Environment Reproduction**: Recreate exact training environments
- **Resource Sharing**: Move GPU workloads between development machines
- **Debug Sessions**: Capture problematic training states

### Production Scenarios
- **Training Migration**: Move long-running training between GPU servers
- **Resource Optimization**: Balance GPU workloads across cluster
- **Fault Recovery**: Resume training after hardware failures
- **Cost Optimization**: Move workloads to cheaper GPU instances

## Verification Steps

### Pre-Checkpoint Validation
```bash
# Verify GPU container is running
nvidia-smi
docker exec gpu-test nvidia-smi

# Check GPU memory usage
nvidia-smi --query-gpu=memory.used --format=csv
```

### Post-Restore Validation
```bash
# Verify restored container has GPU access
docker exec $(docker ps -q -l) nvidia-smi

# Compare GPU memory usage
nvidia-smi --query-gpu=memory.used --format=csv

# Test CUDA functionality
docker exec $(docker ps -q -l) nvidia-smi -L
```

## Troubleshooting

### Common Issues
- **GPU Access Lost**: Container runtime configuration issues
- **CUDA Context Errors**: Driver version mismatches
- **Memory Allocation Failures**: Insufficient GPU memory on target
- **Permission Errors**: GPU device access permissions

### Debug Commands
```bash
# Check NVIDIA runtime configuration
docker info | grep -i nvidia

# Verify GPU device access
ls -la /dev/nvidia*

# Check CUDA driver version
nvidia-smi --query-gpu=driver_version --format=csv
```

This tutorial bridges the gap between GPU computing and container technology, enabling sophisticated AI/ML workflow management.