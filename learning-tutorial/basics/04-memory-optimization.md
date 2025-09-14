# CRIU Pre-Dump Tutorial

## Overview
Pre-dump is an optimization technique that performs an incremental memory dump before the final checkpoint. It reduces downtime by transferring most memory pages in advance while the process continues running.

## How Pre-Dump Works

### Basic Concept
1. **Pre-dump**: Save most memory pages while process runs
2. **Final dump**: Save only changed pages and process state
3. **Result**: Much faster final checkpoint operation

### Implementation (from criu.go:306)
```go
// PreDump does a pre-dump
func (c *Criu) PreDump(ctx context.Context, opts *criu.CriuOpts, nfy Notify) error {
    _, err := c.doSwrk(ctx, criu.CriuReqType_PRE_DUMP, opts, nfy, nil, nil, nil)
    return err
}
```

## When to Use Pre-Dump

### Large Memory Processes
- Applications using several GB of RAM
- Long-running computations with large datasets
- Processes where downtime must be minimized

### Live Migration Scenarios
- Moving critical services between servers
- Minimizing service interruption
- Network bandwidth optimization

## Pre-Dump Process

### Step 1: Initial Pre-Dump
```bash
# Start pre-dump while process runs
cedana pre-dump --pid 1234 --images-dir ./predump-1
```

### Step 2: Optional Additional Pre-Dumps
```bash
# Can do multiple pre-dumps for very large processes
cedana pre-dump --pid 1234 --images-dir ./predump-2 --prev-images-dir ./predump-1
```

### Step 3: Final Dump
```bash
# Fast final dump using pre-dump data
cedana dump --pid 1234 --images-dir ./final-dump --prev-images-dir ./predump-2
```

## Pre-Dump Files
Creates incremental checkpoint files:
- `pages-1.img`: Initial memory pages
- `pagemap-1.img`: Memory page mappings
- `parents`: Links to previous pre-dumps

## Performance Benefits

### Downtime Reduction
- **Without pre-dump**: 30 seconds downtime for 8GB process
- **With pre-dump**: 2-3 seconds downtime for same process

### Network Efficiency
- Transfer bulk data during pre-dump phase
- Final dump transfers minimal changed data
- Reduces migration bandwidth requirements

## Best Practices

### Timing Strategy
1. Start pre-dump during low activity periods
2. Multiple pre-dumps for very large processes
3. Minimize time between final pre-dump and dump

### Resource Management
1. Monitor I/O impact during pre-dump
2. Balance pre-dump frequency vs. performance
3. Consider storage space for multiple pre-dumps

## Limitations
- Requires additional storage for pre-dump images
- More complex checkpoint/restore workflow
- Not beneficial for small processes
- Increased complexity for automation

Pre-dump is essential for minimizing downtime when checkpointing large, memory-intensive applications.