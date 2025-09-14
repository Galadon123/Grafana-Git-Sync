# Web Application Migration

## What We'll Learn
How Cedana migrates running web applications while preserving in-memory state, user sessions, and application data.

## Core Concepts

Web application migration involves preserving:
- **In-Memory State** - Application variables, caches, and session data
- **Active Connections** - HTTP connections and WebSocket sessions
- **Request Context** - In-progress request processing
- **Application State** - Counters, statistics, and runtime data

## Web Application Challenges

### Session Management
- **User Sessions** - Logged-in users and their session state
- **Shopping Carts** - E-commerce temporary data
- **Form State** - Multi-step form progress
- **Authentication Tokens** - JWT tokens and API keys in memory

### Connection State
- **HTTP Keep-Alive** - Persistent HTTP connections
- **WebSocket Connections** - Real-time bidirectional communication
- **Server-Sent Events** - Streaming connections to clients
- **Connection Pools** - Database and external service connections

## Migration Strategy

### Stateless Applications
For stateless web apps:
1. **Simple Migration** - Checkpoint and restore work well
2. **Load Balancer Coordination** - Update routing during migration
3. **Health Checks** - Verify application responds after restore
4. **Connection Recovery** - Clients automatically reconnect

### Stateful Applications
For stateful web apps:
1. **State Externalization** - Move critical state to external stores
2. **Session Store Migration** - Handle Redis/database session stores
3. **Graceful Connection Handling** - Manage active user sessions
4. **State Validation** - Verify application consistency after restore

## Quick Implementation

1. **Identify State** - Determine what application state needs preservation
2. **Prepare Migration** - Coordinate with load balancers and monitoring
3. **Checkpoint Application** - `cedana dump --type containerd --id <web-app>`
4. **Update Infrastructure** - Switch traffic to restored instance
5. **Validate** - Test application functionality and state preservation

## Load Balancer Integration

### Traffic Management
- **Drain Connections** - Stop sending new requests to source
- **Wait for Completion** - Allow in-progress requests to finish
- **Health Check Updates** - Configure health checks for restored instance
- **Gradual Cutover** - Route traffic to new instance

### DNS and Service Discovery
- **DNS Updates** - Update DNS records if needed
- **Service Registry** - Update service discovery systems
- **API Gateway Configuration** - Update routing rules

## Real-World Examples

### Node.js Applications
- Express.js server state and middleware
- In-memory caching and session data
- WebSocket connection management

### Python Web Apps
- Flask/Django application state
- Gunicorn worker process management
- Celery task queue state

### Java Applications
- Spring Boot application context
- Servlet container state
- JVM heap preservation

## Performance Considerations

### Checkpoint Speed
- **Application Size** - Larger applications take longer to checkpoint
- **Memory Usage** - More RAM requires more checkpoint time
- **I/O Patterns** - Applications with heavy I/O may have complex state

### Restore Speed
- **Cold Start vs Warm Restore** - Checkpointed apps start faster than cold starts
- **Dependency Loading** - External dependencies still need initialization
- **Cache Warming** - Application caches need time to repopulate

## Key Files to Explore
- `plugins/containerd/` - Container-level migration logic
- Load balancer and infrastructure coordination happens externally
- Application-specific state handling varies by framework

Web application migration enables zero-downtime deployment patterns and sophisticated load balancing strategies.