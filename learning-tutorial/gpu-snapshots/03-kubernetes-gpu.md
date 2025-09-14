# Tutorial 12: Kubernetes GPU Pod Checkpointing

## Overview
Checkpoint and restore Kubernetes pods that use GPUs, handling GPU resource scheduling, device plugins, and cluster-level coordination.

## Prerequisites
- Completed tutorials 1-3, 6, 8, 11 (Foundation + GPU + K8s + GPU containers)
- Kubernetes cluster with GPU nodes
- NVIDIA device plugin installed
- GPU resource quotas configured

## Key Concepts

### K8s GPU Architecture
- **NVIDIA Device Plugin**: GPU resource discovery and allocation
- **GPU Resource Scheduling**: Pod placement on GPU-enabled nodes
- **Resource Limits**: GPU allocation and sharing policies
- **Node Affinity**: GPU-specific node selection

### Pod GPU Checkpoint Challenges
- **GPU Resource Coordination**: Managing GPU allocation during checkpoint/restore
- **Device Plugin Integration**: Working with Kubernetes device plugins
- **Node Scheduling**: Ensuring GPU availability on target nodes
- **Resource Quota Management**: GPU resource accounting during migration

## What You'll Learn

### GPU Pod Management
- Creating GPU-enabled pods with proper resource requests
- Managing GPU resource limits and sharing
- Using node selectors and affinity for GPU nodes
- Monitoring GPU usage within pods

### Cluster GPU Coordination
- Working with NVIDIA device plugin
- Understanding GPU resource scheduling
- Managing GPU resource quotas
- Handling multi-GPU pods

### Advanced GPU Pod Scenarios
- StatefulSets with GPU persistence
- DaemonSets on GPU nodes
- Job and CronJob GPU workloads
- Multi-container pods with GPU sharing

## Test Lab Setup

### Kubernetes GPU Configuration
```yaml
# Check GPU nodes
kubectl get nodes -l accelerator=nvidia

# Verify NVIDIA device plugin
kubectl get daemonset -n kube-system nvidia-device-plugin-daemonset

# Check available GPU resources
kubectl describe nodes | grep -A 5 nvidia.com/gpu
```

### Simple GPU Pod Test
```yaml
# gpu-test-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-test-pod
spec:
  containers:
  - name: gpu-container
    image: nvidia/cuda:11.8-runtime-ubuntu20.04
    command: ["sh", "-c"]
    args: ["while true; do nvidia-smi; echo 'GPU test running'; sleep 30; done"]
    resources:
      limits:
        nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia
```

```bash
# Deploy and test
kubectl apply -f gpu-test-pod.yaml

# Verify GPU access
kubectl logs gpu-test-pod

# Check GPU resource allocation
kubectl describe pod gpu-test-pod | grep -A 10 Limits
```

### Checkpoint GPU Pod
```bash
# Checkpoint GPU pod
cedana dump --type k8s --namespace default --pod gpu-test-pod --cuda

# Delete original pod
kubectl delete pod gpu-test-pod

# Restore GPU pod
cedana restore --type k8s --checkpoint-dir ./gpu-test-pod-checkpoint

# Verify restored pod has GPU access
kubectl logs gpu-test-pod
kubectl exec gpu-test-pod -- nvidia-smi
```

## AI/ML Training Pod Examples

### TensorFlow Training Job
```yaml
# tf-training-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: tf-gpu-training
spec:
  template:
    spec:
      containers:
      - name: tf-training
        image: tensorflow/tensorflow:latest-gpu
        command: ["python", "-c"]
        args:
          - |
            import tensorflow as tf
            print('GPUs Available:', tf.config.list_physical_devices('GPU'))

            # Simple CNN training for testing
            mnist = tf.keras.datasets.mnist
            (x_train, y_train), (x_test, y_test) = mnist.load_data()
            x_train = x_train.reshape(-1, 28, 28, 1).astype('float32') / 255
            x_test = x_test.reshape(-1, 28, 28, 1).astype('float32') / 255

            model = tf.keras.Sequential([
                tf.keras.layers.Conv2D(32, 3, activation='relu'),
                tf.keras.layers.GlobalAveragePooling2D(),
                tf.keras.layers.Dense(10, activation='softmax')
            ])

            model.compile(optimizer='adam',
                         loss='sparse_categorical_crossentropy',
                         metrics=['accuracy'])

            model.fit(x_train, y_train, epochs=10, validation_data=(x_test, y_test))
        resources:
          limits:
            nvidia.com/gpu: 1
            memory: 4Gi
          requests:
            nvidia.com/gpu: 1
            memory: 2Gi
      restartPolicy: Never
  backoffLimit: 3
```

```bash
# Deploy training job
kubectl apply -f tf-training-job.yaml

# Monitor training progress
kubectl logs -f job/tf-gpu-training

# Checkpoint during training
JOB_POD=$(kubectl get pods -l job-name=tf-gpu-training -o jsonpath='{.items[0].metadata.name}')
cedana dump --type k8s --namespace default --pod $JOB_POD --cuda
```

### Multi-GPU PyTorch Training
```yaml
# pytorch-multigpu-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pytorch-multigpu
spec:
  containers:
  - name: pytorch-training
    image: pytorch/pytorch:latest
    command: ["python", "-c"]
    args:
      - |
        import torch
        import torch.nn as nn
        import torch.distributed as dist

        print(f'CUDA available: {torch.cuda.is_available()}')
        print(f'GPU count: {torch.cuda.device_count()}')

        if torch.cuda.device_count() > 1:
            # Multi-GPU training setup
            model = nn.DataParallel(nn.Linear(100, 1).cuda())
            print(f'Using {torch.cuda.device_count()} GPUs')
        else:
            model = nn.Linear(100, 1).cuda()
            print('Using single GPU')

        # Simple training loop
        optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
        for epoch in range(1000):
            x = torch.randn(64, 100).cuda()
            y = torch.randn(64, 1).cuda()

            pred = model(x)
            loss = nn.functional.mse_loss(pred, y)

            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

            if epoch % 100 == 0:
                print(f'Epoch {epoch}: Loss = {loss.item():.4f}')
    resources:
      limits:
        nvidia.com/gpu: 2
        memory: 8Gi
      requests:
        nvidia.com/gpu: 2
        memory: 4Gi
  nodeSelector:
    accelerator: nvidia
```

## StatefulSet GPU Applications

### Persistent GPU Training
```yaml
# gpu-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: gpu-training-stateful
spec:
  serviceName: gpu-training
  replicas: 1
  selector:
    matchLabels:
      app: gpu-training
  template:
    metadata:
      labels:
        app: gpu-training
    spec:
      containers:
      - name: training
        image: tensorflow/tensorflow:latest-gpu
        volumeMounts:
        - name: training-data
          mountPath: /data
        - name: model-checkpoints
          mountPath: /checkpoints
        resources:
          limits:
            nvidia.com/gpu: 1
        command: ["python", "-c"]
        args:
          - |
            import os
            import tensorflow as tf

            # Use persistent storage for model checkpoints
            checkpoint_dir = '/checkpoints'
            os.makedirs(checkpoint_dir, exist_ok=True)

            # Training with periodic model saving
            model = tf.keras.Sequential([tf.keras.layers.Dense(1)])
            model.compile(optimizer='adam', loss='mse')

            checkpoint_callback = tf.keras.callbacks.ModelCheckpoint(
                filepath=os.path.join(checkpoint_dir, 'model_{epoch:02d}.h5'),
                save_freq='epoch'
            )

            # Generate training data
            x = tf.random.normal([1000, 10])
            y = tf.random.normal([1000, 1])

            model.fit(x, y, epochs=100, callbacks=[checkpoint_callback])
  volumeClaimTemplates:
  - metadata:
      name: training-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
  - metadata:
      name: model-checkpoints
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

## Verification and Monitoring

### GPU Resource Monitoring
```bash
# Check GPU usage across cluster
kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, gpu_capacity: .status.capacity["nvidia.com/gpu"], gpu_allocatable: .status.allocatable["nvidia.com/gpu"]}'

# Monitor GPU pod resource usage
kubectl top pod --containers

# Check GPU device allocation
kubectl describe nodes | grep -A 10 "nvidia.com/gpu"
```

### Post-Checkpoint Validation
```bash
# Verify restored pod GPU access
kubectl exec gpu-test-pod -- nvidia-smi

# Check GPU memory usage
kubectl exec gpu-test-pod -- nvidia-smi --query-gpu=memory.used --format=csv

# Validate training continuation (for training pods)
kubectl logs gpu-test-pod | tail -20
```

## Advanced Scenarios

### Cross-Node GPU Migration
```bash
# Checkpoint pod on one GPU node
cedana dump --type k8s --pod gpu-workload --node gpu-node-1 --cuda

# Restore on different GPU node
cedana restore --type k8s --checkpoint-dir ./gpu-workload --target-node gpu-node-2
```

### GPU Resource Scaling
```bash
# Checkpoint single-GPU pod
cedana dump --type k8s --pod single-gpu-app --cuda

# Restore with different GPU allocation
cedana restore --type k8s --checkpoint-dir ./single-gpu-app --gpu-count 2
```

This tutorial enables sophisticated GPU workload management in Kubernetes environments, supporting advanced AI/ML operations at scale.