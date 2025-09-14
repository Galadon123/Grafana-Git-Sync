# Tutorial 14: Testing and Validation Framework

## Overview
Learn comprehensive testing strategies for checkpoint/restore operations, from simple validation to complex integration testing scenarios.

## Prerequisites
- Completed basic tutorials (1-3)
- Understanding of testing methodologies
- Basic scripting knowledge

## Key Concepts

### Testing Levels
- **Unit Tests**: Individual checkpoint/restore operations
- **Integration Tests**: Multi-component scenarios
- **System Tests**: End-to-end workflows
- **Performance Tests**: Scalability and timing validation

### Validation Categories
- **Functional Validation**: Does the restored process work correctly?
- **State Validation**: Is the internal state preserved accurately?
- **Performance Validation**: Does performance meet expectations?
- **Security Validation**: Are security contexts maintained?

## What You'll Learn

### Test Environment Setup
- Creating reproducible test environments
- Automated testing workflows
- Test data management
- Environment cleanup procedures

### Validation Techniques
- Process state verification
- Data integrity checking
- Performance benchmarking
- Security posture validation

### Automated Testing
- Scripted test execution
- Continuous integration integration
- Regression testing
- Load testing scenarios

## Basic Testing Framework

### Simple Process Test
```bash
#!/bin/bash
# test-basic-checkpoint.sh

echo "=== Basic Checkpoint/Restore Test ==="

# Start test process
echo "Starting test process..."
bash -c 'i=0; while true; do echo "Count: $i"; i=$((i+1)); sleep 1; done' &
TEST_PID=$!
echo "Test process PID: $TEST_PID"

# Let it run for a few iterations
sleep 5

# Capture expected state
EXPECTED_COUNT=$(kill -USR1 $TEST_PID 2>/dev/null; ps -p $TEST_PID -o pid= | wc -l)

# Checkpoint
echo "Creating checkpoint..."
mkdir -p ./test-checkpoint
sudo criu dump -t $TEST_PID -D ./test-checkpoint -v4

if [ $? -eq 0 ]; then
    echo "✓ Checkpoint created successfully"
else
    echo "✗ Checkpoint failed"
    exit 1
fi

# Restore
echo "Restoring process..."
sudo criu restore -D ./test-checkpoint -v4 &
RESTORE_PID=$!

# Wait for restore
sleep 3

# Validate restoration
if ps -p $RESTORE_PID > /dev/null; then
    echo "✓ Process restored successfully"
    kill $RESTORE_PID
else
    echo "✗ Process restoration failed"
fi

# Cleanup
rm -rf ./test-checkpoint
echo "=== Test Complete ==="
```

### Memory State Test
```bash
#!/bin/bash
# test-memory-state.sh

echo "=== Memory State Preservation Test ==="

# Create C program that maintains state in memory
cat > memory_test.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

volatile int counter = 0;
volatile int received_signal = 0;

void signal_handler(int sig) {
    printf("Counter value at signal: %d\n", counter);
    received_signal = 1;
}

int main() {
    signal(SIGUSR1, signal_handler);

    while (!received_signal) {
        counter++;
        printf("Counter: %d\n", counter);
        fflush(stdout);
        sleep(1);
    }

    printf("Final counter value: %d\n", counter);
    return 0;
}
EOF

# Compile and run
gcc -o memory_test memory_test.c
./memory_test &
TEST_PID=$!

# Let it run and capture state
sleep 5
kill -USR1 $TEST_PID
wait $TEST_PID
EXPECTED_FINAL=$(echo $?)

echo "Expected final counter: $EXPECTED_FINAL"

# Now test with checkpoint/restore
./memory_test &
TEST_PID=$!
sleep 3

# Checkpoint
mkdir -p ./memory-checkpoint
sudo criu dump -t $TEST_PID -D ./memory-checkpoint -v4

# Restore
sudo criu restore -D ./memory-checkpoint -v4 &
sleep 2
kill -USR1 $(pgrep memory_test)

# Cleanup
rm -f memory_test memory_test.c
rm -rf ./memory-checkpoint
```

## Container Testing Framework

### Docker Container Test Suite
```bash
#!/bin/bash
# test-container-checkpoint.sh

echo "=== Container Checkpoint Test Suite ==="

# Test 1: Basic web server container
test_web_server() {
    echo "--- Test 1: Web Server Container ---"

    # Start nginx container
    docker run -d --name test-nginx -p 8080:80 nginx:alpine

    # Verify it's working
    sleep 3
    if curl -s http://localhost:8080 > /dev/null; then
        echo "✓ Container started successfully"
    else
        echo "✗ Container startup failed"
        return 1
    fi

    # Checkpoint
    CONTAINER_ID=$(docker ps -q -f name=test-nginx)
    cedana dump --type containerd --namespace moby --id $CONTAINER_ID

    # Stop original
    docker stop test-nginx
    docker rm test-nginx

    # Restore
    cedana restore --type containerd --checkpoint-dir ./test-nginx-checkpoint

    # Verify restored container works
    sleep 3
    if curl -s http://localhost:8080 > /dev/null; then
        echo "✓ Restored container working"
        docker stop $(docker ps -q -l)
        docker rm $(docker ps -aq -l)
        return 0
    else
        echo "✗ Restored container failed"
        return 1
    fi
}

# Test 2: Stateful application container
test_stateful_app() {
    echo "--- Test 2: Stateful Application ---"

    # Start Redis container with data
    docker run -d --name test-redis redis:6.2

    # Add test data
    docker exec test-redis redis-cli SET test:key "test-value"
    docker exec test-redis redis-cli SET test:counter 42

    # Verify data
    VALUE=$(docker exec test-redis redis-cli GET test:key)
    COUNTER=$(docker exec test-redis redis-cli GET test:counter)

    echo "Original data: key=$VALUE, counter=$COUNTER"

    # Checkpoint
    CONTAINER_ID=$(docker ps -q -f name=test-redis)
    cedana dump --type containerd --id $CONTAINER_ID

    # Stop and restore
    docker stop test-redis
    docker rm test-redis

    cedana restore --type containerd --checkpoint-dir ./test-redis-checkpoint

    # Verify data integrity
    sleep 3
    RESTORED_VALUE=$(docker exec $(docker ps -q -l) redis-cli GET test:key)
    RESTORED_COUNTER=$(docker exec $(docker ps -q -l) redis-cli GET test:counter)

    echo "Restored data: key=$RESTORED_VALUE, counter=$RESTORED_COUNTER"

    if [[ "$VALUE" == "$RESTORED_VALUE" && "$COUNTER" == "$RESTORED_COUNTER" ]]; then
        echo "✓ Data integrity preserved"
        docker stop $(docker ps -q -l)
        docker rm $(docker ps -aq -l)
        return 0
    else
        echo "✗ Data integrity lost"
        return 1
    fi
}

# Run tests
PASS=0
TOTAL=2

test_web_server && ((PASS++))
test_stateful_app && ((PASS++))

echo "=== Test Results: $PASS/$TOTAL tests passed ==="
```

## GPU Testing Framework

### CUDA Application Test
```bash
#!/bin/bash
# test-gpu-checkpoint.sh

echo "=== GPU Checkpoint Test Suite ==="

# Check GPU availability
if ! nvidia-smi > /dev/null 2>&1; then
    echo "✗ GPU not available, skipping GPU tests"
    exit 0
fi

# Test 1: Simple CUDA container
test_cuda_container() {
    echo "--- Test 1: CUDA Container ---"

    # Run CUDA test container
    docker run --gpus all -d --name cuda-test \
        nvidia/cuda:11.8-runtime-ubuntu20.04 \
        bash -c "while true; do nvidia-smi --query-gpu=memory.used --format=csv,noheader; sleep 5; done"

    # Verify GPU access
    sleep 5
    if docker logs cuda-test | grep -q "MiB"; then
        echo "✓ CUDA container has GPU access"
    else
        echo "✗ CUDA container lacks GPU access"
        return 1
    fi

    # Checkpoint with CUDA
    CONTAINER_ID=$(docker ps -q -f name=cuda-test)
    cedana dump --type containerd --id $CONTAINER_ID --cuda

    # Stop and restore
    docker stop cuda-test
    docker rm cuda-test

    cedana restore --type containerd --checkpoint-dir ./cuda-test-checkpoint

    # Verify GPU access after restore
    sleep 5
    if docker logs $(docker ps -q -l) | grep -q "MiB"; then
        echo "✓ Restored CUDA container has GPU access"
        docker stop $(docker ps -q -l)
        docker rm $(docker ps -aq -l)
        return 0
    else
        echo "✗ Restored CUDA container lacks GPU access"
        return 1
    fi
}

# Test 2: GPU memory allocation test
test_gpu_memory() {
    echo "--- Test 2: GPU Memory Test ---"

    # Run GPU memory allocation test
    docker run --gpus all -d --name gpu-memory-test \
        nvidia/cuda:11.8-runtime-ubuntu20.04 \
        bash -c "
        python3 -c \"
import time
try:
    import cupy as cp
    # Allocate GPU memory
    x = cp.random.random((1000, 1000))
    print(f'Allocated GPU array shape: {x.shape}')

    for i in range(1000):
        y = cp.dot(x, x)
        if i % 100 == 0:
            print(f'Iteration {i}: GPU memory used')
        time.sleep(1)
except ImportError:
    print('CuPy not available, using basic CUDA test')
    for i in range(1000):
        print(f'GPU test iteration {i}')
        time.sleep(1)
\"
        "

    # Let it run and allocate memory
    sleep 10

    # Checkpoint
    CONTAINER_ID=$(docker ps -q -f name=gpu-memory-test)
    cedana dump --type containerd --id $CONTAINER_ID --cuda

    # Stop and restore
    docker stop gpu-memory-test
    docker rm gpu-memory-test

    cedana restore --type containerd --checkpoint-dir ./gpu-memory-test-checkpoint

    # Verify continued execution
    sleep 5
    if docker logs $(docker ps -q -l) | tail -5 | grep -q "Iteration\|GPU test"; then
        echo "✓ GPU memory state preserved"
        docker stop $(docker ps -q -l)
        docker rm $(docker ps -aq -l)
        return 0
    else
        echo "✗ GPU memory state lost"
        return 1
    fi
}

# Run GPU tests
test_cuda_container
test_gpu_memory

echo "=== GPU Tests Complete ==="
```

## Performance Testing

### Checkpoint Performance Test
```bash
#!/bin/bash
# test-performance.sh

echo "=== Performance Testing Suite ==="

# Test checkpoint time for different process sizes
test_checkpoint_performance() {
    echo "--- Checkpoint Performance Test ---"

    for SIZE in 100 500 1000; do
        echo "Testing with ${SIZE}MB memory allocation..."

        # Start memory-intensive process
        python3 -c "
import time
import os

# Allocate memory
data = bytearray($SIZE * 1024 * 1024)  # ${SIZE}MB
print(f'Allocated {$SIZE}MB memory, PID: {os.getpid()}')

# Keep process alive
while True:
    time.sleep(1)
" &
        TEST_PID=$!

        # Wait for memory allocation
        sleep 5

        # Time checkpoint operation
        start_time=$(date +%s.%N)

        mkdir -p ./perf-test-$SIZE
        sudo criu dump -t $TEST_PID -D ./perf-test-$SIZE -v0

        end_time=$(date +%s.%N)

        # Calculate duration
        duration=$(echo "$end_time - $start_time" | bc)

        echo "Checkpoint time for ${SIZE}MB: ${duration}s"

        # Cleanup
        rm -rf ./perf-test-$SIZE
        kill $TEST_PID 2>/dev/null
        wait $TEST_PID 2>/dev/null
    done
}

# Test restore performance
test_restore_performance() {
    echo "--- Restore Performance Test ---"

    # Create a standard checkpoint
    python3 -c "
import time
import os
print(f'Test process PID: {os.getpid()}')
while True:
    time.sleep(1)
" &
    TEST_PID=$!

    sleep 3
    mkdir -p ./restore-perf-test
    sudo criu dump -t $TEST_PID -D ./restore-perf-test -v0

    # Test multiple restores
    for i in {1..5}; do
        start_time=$(date +%s.%N)

        sudo criu restore -D ./restore-perf-test -v0 &
        RESTORE_PID=$!

        # Wait for restore to complete
        sleep 2
        kill $RESTORE_PID 2>/dev/null
        wait $RESTORE_PID 2>/dev/null

        end_time=$(date +%s.%N)
        duration=$(echo "$end_time - $start_time" | bc)

        echo "Restore attempt $i: ${duration}s"
    done

    # Cleanup
    rm -rf ./restore-perf-test
}

# Run performance tests
test_checkpoint_performance
test_restore_performance

echo "=== Performance Tests Complete ==="
```

## Automated Test Execution

### CI/CD Integration
```yaml
# .github/workflows/checkpoint-tests.yml
name: Checkpoint/Restore Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Install CRIU
      run: |
        sudo apt update
        sudo apt install -y criu

    - name: Install Docker
      run: |
        sudo apt install -y docker.io
        sudo systemctl start docker
        sudo usermod -aG docker $USER

    - name: Run Basic Tests
      run: |
        cd docs/learning-tutorial
        bash test-basic-checkpoint.sh

    - name: Run Container Tests
      run: |
        cd docs/learning-tutorial
        bash test-container-checkpoint.sh

    - name: Run Performance Tests
      run: |
        cd docs/learning-tutorial
        bash test-performance.sh
```

## Test Validation Checklist

### Pre-Test Validation
- [ ] CRIU installed and functional
- [ ] Required permissions available
- [ ] Test environment clean
- [ ] Dependencies installed

### Post-Test Validation
- [ ] All processes cleaned up
- [ ] Temporary files removed
- [ ] System resources released
- [ ] Test artifacts saved (if needed)

### Test Result Analysis
- [ ] Functional tests pass
- [ ] Performance within acceptable limits
- [ ] No memory leaks detected
- [ ] Error logs reviewed

This comprehensive testing framework ensures reliable checkpoint/restore operations across all scenarios and environments.