# CRIU Page Server Tutorial

## Overview
Page server enables lazy migration by serving memory pages on-demand over the network. Instead of transferring all memory during checkpoint, pages are fetched when the restored process actually accesses them.

## How Page Server Works

### Traditional vs Lazy Migration
- **Traditional**: Transfer all memory pages during restore
- **Lazy**: Transfer only essential pages, fetch others on-demand
- **Result**: Much faster restore startup time

### Implementation (from criu.go:312)
```go
// StartPageServer starts the page server
func (c *Criu) StartPageServer(ctx context.Context, opts *criu.CriuOpts) error {
    _, err := c.doSwrk(ctx, criu.CriuReqType_PAGE_SERVER, opts, nil, nil, nil, nil)
    return err
}

// StartPageServerChld starts the page server and returns PID and port
func (c *Criu) StartPageServerChld(ctx context.Context, opts *criu.CriuOpts) (int, int, error) {
    resp, err := c.doSwrkWithResp(ctx, criu.CriuReqType_PAGE_SERVER_CHLD, opts, nil, nil, nil, nil, nil)
    return int(resp.GetPs().GetPid()), int(resp.GetPs().GetPort()), nil
}
```

## Page Server Setup

### Source Machine (where checkpoint exists)
```bash
# Start page server on source machine
cedana page-server --images-dir ./checkpoint-images --port 9999
```

### Target Machine (where process will restore)
```bash
# Restore with lazy migration
cedana restore --images-dir ./checkpoint-images \
    --page-server 192.168.1.100:9999 \
    --lazy-pages
```

## Use Cases

### Fast Service Startup
- Web services that need quick startup
- Applications with large but sparsely accessed memory
- Containers with large base images

### Network-Efficient Migration
- Slow network connections
- Cross-datacenter migration
- Bandwidth-constrained environments

### Memory-Constrained Environments
- Target machine has less memory than source
- On-demand memory allocation
- Gradual memory warming

## Page Server Architecture

### Server Side
1. Serves memory pages from checkpoint images
2. Tracks which pages have been requested
3. Handles multiple client connections
4. Provides page-fault resolution

### Client Side (Restored Process)
1. Starts with minimal memory footprint
2. Generates page faults on memory access
3. Requests pages from server as needed
4. Caches frequently accessed pages locally

## Performance Characteristics

### Startup Time
- **Traditional restore**: Wait for all memory transfer
- **Lazy restore**: Start immediately with minimal pages
- **Trade-off**: Slower initial execution due to page faults

### Memory Usage
- Lower initial memory footprint
- Gradual increase as pages are accessed
- Better memory utilization efficiency

### Network Pattern
- Bursty network traffic during page faults
- Decreases over time as working set is cached
- Total transfer may be less than full dump

## Configuration Options

### Page Server Settings
```go
opts.PageServer = proto.Bool(true)
opts.PageServerAddress = proto.String("0.0.0.0")
opts.PageServerPort = proto.Int32(9999)
```

### Lazy Pages Settings
```go
opts.LazyPages = proto.Bool(true)
opts.PageServerAddress = proto.String("source-server:9999")
```

## Best Practices

### Network Setup
1. Ensure stable network connection between machines
2. Use dedicated network for page server traffic
3. Monitor network latency and bandwidth

### Monitoring
1. Track page fault frequency
2. Monitor network utilization
3. Measure application performance impact

### Security
1. Secure page server communication
2. Restrict page server access
3. Use VPN for cross-network migration

## Limitations
- Requires network connectivity during execution
- Page faults can impact performance
- Complex setup compared to traditional migration
- Network failures can crash restored process

Page server enables innovative migration patterns, especially useful for large applications where quick startup is more important than initial performance.