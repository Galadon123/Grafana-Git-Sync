# Page Server Implementation

## What We'll Learn
How Cedana enables lazy migration using page servers to serve memory pages on-demand over the network.

## Core Concepts

Traditional migration transfers all memory during restore, which can be slow. Page server enables "lazy migration":
- **On-Demand Loading** - Only fetch memory pages when the process actually needs them
- **Fast Startup** - Process starts quickly with minimal initial memory
- **Network Efficiency** - Transfer only accessed pages, not entire memory space
- **Gradual Warming** - Memory usage grows as application runs

## How Page Server Works

The lazy migration process:
1. **Server Setup** - Start page server on source machine with checkpoint files
2. **Quick Restore** - Target machine starts process with minimal memory
3. **Page Faults** - When process accesses missing memory, generates page fault
4. **On-Demand Fetch** - Page server sends requested pages over network
5. **Local Caching** - Frequently used pages cached locally

## When to Use Page Server

Lazy migration is beneficial for:
- **Large Memory Applications** - Apps with GB of memory but sparse access patterns
- **Network-Limited Migration** - Slow connections between source and target
- **Quick Failover** - Need fast startup time more than initial performance
- **Memory-Constrained Targets** - Target has less memory than source

## Quick Implementation

1. **Start Page Server** - `cedana page-server --images-dir ./checkpoint --port 9999`
2. **Lazy Restore** - `cedana restore --page-server source-host:9999 --lazy-pages`
3. **Monitor Usage** - Watch network traffic and page fault patterns
4. **Performance Trade-off** - Slower initial access, faster startup

## Key Files to Explore
- `pkg/criu/criu.go` - `StartPageServer()` and `StartPageServerChld()` functions
- CRIU's page server implementation handles network communication

Page servers enable innovative migration strategies where startup speed matters more than initial performance.