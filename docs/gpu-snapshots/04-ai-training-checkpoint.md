# AI Training Checkpointing

## What We'll Learn
How Cedana enables checkpointing of long-running AI/ML training jobs, preserving model state, optimizer state, and training progress.

## Core Concepts

AI training checkpointing involves multiple state layers:
- **Model State** - Neural network weights and parameters in GPU memory
- **Optimizer State** - Adam, SGD, or other optimizer internal state
- **Training Loop State** - Current epoch, batch index, learning rate schedules
- **Data Pipeline State** - Dataset position, data loader state, augmentation state

## Training State Components

### Model and Optimizer GPU State
- **Parameter Tensors** - Model weights stored in GPU memory
- **Gradient Buffers** - Computed gradients awaiting optimizer updates
- **Momentum Buffers** - Optimizer momentum terms (Adam, RMSprop)
- **Running Statistics** - BatchNorm running means and variances

### Training Progress State
- **Epoch Counter** - Current training epoch number
- **Global Step** - Total number of training steps completed
- **Learning Rate** - Current learning rate value and schedule state
- **Loss History** - Recent loss values and validation metrics

## Framework Integration

### PyTorch Training
- **Model State Dict** - PyTorch model parameters
- **Optimizer State Dict** - Optimizer internal state
- **Learning Rate Scheduler** - Schedule state and current values
- **Random Number Generators** - CUDA and CPU RNG states for reproducibility

### TensorFlow Training
- **Model Checkpoints** - TensorFlow SavedModel or checkpoint format
- **Variable State** - TensorFlow variable values in GPU memory
- **Optimizer Variables** - Internal optimizer state variables
- **Dataset State** - tf.data dataset iterator state

## Checkpoint Strategy

### Application-Level vs System-Level
- **Application Checkpointing** - Training script saves model/optimizer state
- **System Checkpointing** - Cedana captures entire process GPU state
- **Hybrid Approach** - Combine both for comprehensive state preservation

### Timing Considerations
- **Epoch Boundaries** - Natural checkpoint points between epochs
- **Step Intervals** - Regular checkpointing every N training steps
- **Time-Based** - Checkpoint every hour or based on wall-clock time
- **Memory Usage** - Checkpoint when GPU memory usage is stable

## Quick Implementation

1. **Training Setup** - Start AI training job with GPU acceleration
2. **State Monitoring** - Monitor training progress and GPU memory usage
3. **Checkpoint Timing** - Choose optimal checkpoint timing (epoch end)
4. **System Checkpoint** - `cedana dump --type process --pid <training-pid> --cuda`
5. **Resume Training** - Restore and verify training continues from checkpoint

## Long-Running Training Benefits

### Training Continuity
- **Hardware Failures** - Resume training after GPU or system failures
- **Resource Migration** - Move training between different GPU systems
- **Maintenance Windows** - Checkpoint before system maintenance
- **Cost Optimization** - Move training to cheaper GPU instances

### Experiment Management
- **Reproducible Results** - Exact state preservation enables reproducibility
- **Hyperparameter Exploration** - Branch training from checkpoints with different parameters
- **Model Comparison** - Compare models trained from same checkpoint point
- **Debug Analysis** - Analyze training state at specific points

## Multi-GPU Training

### Distributed Training State
- **Model Replicas** - Multiple model copies across GPUs
- **Gradient Synchronization** - AllReduce communication state
- **Data Parallel State** - Data distribution across workers
- **Process Group State** - Distributed communication group membership

### Coordination Challenges
- **Synchronization Points** - All workers must checkpoint simultaneously
- **State Consistency** - Ensure consistent state across all workers
- **Communication State** - NCCL or other communication library state
- **Load Balancing** - Maintain even workload distribution after restore

## Performance Considerations

### Checkpoint Overhead
- **GPU Memory Size** - Larger models take longer to checkpoint
- **I/O Bandwidth** - Storage speed affects checkpoint duration
- **Training Interruption** - Balance checkpoint frequency vs training efficiency
- **Network Transfer** - Consider data transfer time for remote migration

### Restore Performance
- **GPU Warming** - Restored GPU memory may need cache warming
- **Model Compilation** - Some frameworks recompile models after restore
- **Data Pipeline** - Dataset loading and preprocessing restart
- **Validation** - Post-restore model validation and sanity checks

## Key Files to Explore
- `internal/cedana/criu/checks_cuda.go` - GPU state validation for training jobs
- Framework-specific checkpointing APIs (PyTorch, TensorFlow)
- Training script integration with system-level checkpointing

AI training checkpointing enables robust, fault-tolerant machine learning workflows with complete state preservation.