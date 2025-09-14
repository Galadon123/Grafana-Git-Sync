# Stateful Application Handling

## What We'll Learn
How Cedana checkpoints containers running databases, caches, and other stateful applications while maintaining data consistency.

## Core Concepts

Stateful applications require special care during checkpointing because they have:
- **Persistent Data** - Information that must survive container restarts
- **Transaction State** - In-progress operations that affect data integrity
- **Connection State** - Active client connections and sessions
- **Lock Management** - Database locks and concurrent access coordination

## Database-Specific Challenges

### Transaction Consistency
- **Active Transactions** - In-progress operations during checkpoint
- **Write-Ahead Logs** - Transaction logs that ensure consistency
- **Lock States** - Database table and row locks
- **Buffer Management** - Dirty pages in memory not yet written to disk

### Connection Handling
- **Client Sessions** - Active database connections from applications
- **Connection Pools** - Managed connection resources
- **Authentication State** - User sessions and security contexts
- **Prepared Statements** - Pre-compiled SQL statements in memory

## Checkpoint Preparation

### Database Preparation
1. **Flush Buffers** - Ensure all dirty pages written to disk
2. **Complete Transactions** - Wait for or abort active transactions
3. **Checkpoint Database** - Use database-specific checkpoint commands
4. **Lock Coordination** - Handle or release problematic locks

### Application Coordination
- **Graceful Shutdown** - Allow applications to close connections cleanly
- **State Validation** - Verify data consistency before checkpoint
- **Backup Coordination** - Coordinate with existing backup schedules

## Quick Implementation

1. **Prepare Database** - Flush buffers and handle transactions
2. **Coordinate Connections** - Manage client connection state
3. **Checkpoint Container** - `cedana dump --type containerd --id <db-container>`
4. **Validate Restore** - Verify data integrity and connection handling

## Common Stateful Applications

### PostgreSQL Containers
- Handle WAL (Write-Ahead Log) state
- Manage connection pooler state
- Coordinate replication if applicable

### Redis Containers
- Handle in-memory dataset and persistence
- Manage pub/sub subscription state
- Coordinate cluster member relationships

### MySQL Containers
- Handle InnoDB transaction logs
- Manage replication state
- Coordinate connection thread state

## Restore Considerations

### Data Integrity
- **Consistency Checks** - Verify database integrity after restore
- **Recovery Procedures** - Handle any corruption detected
- **Index Rebuilding** - Repair any damaged indexes

### Connection Recovery
- **Client Reconnection** - Applications must reconnect to database
- **Session State Loss** - Temporary sessions will be lost
- **Connection Pool Rebuild** - Connection pools need reinitialization

## Key Files to Explore
- Cedana's container plugins handle process-level checkpointing
- Database-specific logic typically handled at application layer
- Container runtime coordination in `plugins/containerd/`

Stateful application checkpointing requires coordination between Cedana's container capabilities and application-specific consistency requirements.