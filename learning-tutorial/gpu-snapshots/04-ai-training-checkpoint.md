# GPU AI Training Checkpoint

## What You'll Learn
Checkpoint long-running AI training jobs to save progress and resume on different machines.

## Lab Setup (10 minutes)
```bash
# Install PyTorch with CUDA
pip install torch torchvision

# Create simple training script
cat > train_model.py << EOF
import torch
import time
import os

print(f"CUDA available: {torch.cuda.is_available()}")
print(f"GPU count: {torch.cuda.device_count()}")

# Simple neural network training
model = torch.nn.Linear(100, 1).cuda()
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)

for epoch in range(10000):
    x = torch.randn(32, 100).cuda()
    y = torch.randn(32, 1).cuda()

    loss = torch.nn.functional.mse_loss(model(x), y)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

    if epoch % 500 == 0:
        print(f"Epoch {epoch}: Loss = {loss.item():.4f}")

    time.sleep(0.1)  # Simulate training time
EOF
```

## Run Training
```bash
# Start training
python train_model.py &
TRAIN_PID=$!
echo "Training PID: $TRAIN_PID"

# Let it train for a few epochs
sleep 10
```

## Checkpoint Training
```bash
# Checkpoint the training process
sudo criu dump -t $TRAIN_PID -D ./training-checkpoint --cuda -v4

# Kill original training
kill $TRAIN_PID
```

## Restore Training
```bash
# Restore training on same or different GPU machine
sudo criu restore -D ./training-checkpoint -v4

# Training continues from where it left off!
```

## What Happens
- GPU memory (model weights, gradients) preserved
- CUDA contexts maintained
- Training resumes exactly where it stopped
- Works across compatible GPUs

## Test Success
✅ Training continues with same loss values
✅ GPU memory usage restored
✅ No CUDA errors after restore

This enables moving long AI training jobs between GPU servers!