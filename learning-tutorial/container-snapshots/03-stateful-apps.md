# Tutorial 13: Database Container Checkpointing

## Overview
Learn to checkpoint and restore containerized databases while maintaining data consistency, handling transactions, and coordinating with application workloads.

## Prerequisites
- Completed tutorials 1-3, 7 (Foundation + Container basics)
- Understanding of database fundamentals
- Docker and basic SQL knowledge

## Key Concepts

### Database Checkpoint Challenges
- **Transaction Consistency**: Ensuring ACID properties during checkpoint
- **Lock Management**: Handling database locks and connections
- **Log File Coordination**: Managing transaction logs and write-ahead logs
- **Data Integrity**: Preventing corruption during checkpoint/restore

### Stateful Container Considerations
- **Persistent Storage**: Volume management and data persistence
- **Connection Handling**: Client connection state and recovery
- **Replication Coordination**: Master-slave and cluster configurations
- **Backup Integration**: Coordinating with existing backup strategies

## What You'll Learn

### Database-Specific Techniques
- Preparing databases for checkpoint operations
- Coordinating checkpoint timing with transaction boundaries
- Managing database connections during migration
- Validating data integrity after restore

### Storage and Persistence
- Checkpoint storage volumes and bind mounts
- Handling database file consistency
- Managing temporary files and logs
- Coordinating with persistent volume claims

## Test Lab Setup

### PostgreSQL Container Test
```bash
# Create data directory
mkdir -p ~/db-lab/postgres-data

# Run PostgreSQL container
docker run -d --name postgres-test \
    -e POSTGRES_PASSWORD=testpass \
    -e POSTGRES_DB=testdb \
    -v ~/db-lab/postgres-data:/var/lib/postgresql/data \
    -p 5432:5432 \
    postgres:13

# Wait for startup
sleep 10

# Create test data
docker exec postgres-test psql -U postgres -d testdb -c "
CREATE TABLE test_data (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    value INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);

INSERT INTO test_data (name, value)
SELECT 'test_' || i, i
FROM generate_series(1, 1000) i;
"

# Verify data
docker exec postgres-test psql -U postgres -d testdb -c "SELECT COUNT(*) FROM test_data;"
```

### Checkpoint PostgreSQL
```bash
# Flush database buffers before checkpoint
docker exec postgres-test psql -U postgres -d testdb -c "CHECKPOINT;"

# Create checkpoint
cedana dump --type containerd --namespace moby --id $(docker ps -q -f name=postgres-test)

# Stop original container
docker stop postgres-test
docker rm postgres-test

# Restore container
cedana restore --type containerd --checkpoint-dir ./postgres-test-checkpoint

# Verify data integrity
docker exec $(docker ps -q -l) psql -U postgres -d testdb -c "SELECT COUNT(*) FROM test_data;"
```

## Database-Specific Examples

### MySQL Container Checkpoint
```bash
# Run MySQL container
docker run -d --name mysql-test \
    -e MYSQL_ROOT_PASSWORD=rootpass \
    -e MYSQL_DATABASE=testdb \
    -e MYSQL_USER=testuser \
    -e MYSQL_PASSWORD=testpass \
    -v ~/db-lab/mysql-data:/var/lib/mysql \
    mysql:8.0

# Create test data
docker exec mysql-test mysql -u root -prootpass testdb -e "
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username, email) VALUES
('user1', 'user1@example.com'),
('user2', 'user2@example.com'),
('user3', 'user3@example.com');
"

# Checkpoint with transaction flush
docker exec mysql-test mysql -u root -prootpass -e "FLUSH TABLES WITH READ LOCK;"
cedana dump --type containerd --id $(docker ps -q -f name=mysql-test)
docker exec mysql-test mysql -u root -prootpass -e "UNLOCK TABLES;"
```

### Redis Container Checkpoint
```bash
# Run Redis container
docker run -d --name redis-test \
    -v ~/db-lab/redis-data:/data \
    redis:6.2 redis-server --save 60 1 --appendonly yes

# Add test data
docker exec redis-test redis-cli SET user:1 "John Doe"
docker exec redis-test redis-cli SET user:2 "Jane Smith"
docker exec redis-test redis-cli LPUSH queue:tasks "task1" "task2" "task3"
docker exec redis-test redis-cli HSET product:1 name "Widget" price "19.99"

# Force data persistence before checkpoint
docker exec redis-test redis-cli BGSAVE
docker exec redis-test redis-cli BGREWRITEAOF

# Wait for background operations
sleep 5

# Checkpoint Redis
cedana dump --type containerd --id $(docker ps -q -f name=redis-test)

# Verify after restore
docker exec $(docker ps -q -l) redis-cli GET user:1
docker exec $(docker ps -q -l) redis-cli LRANGE queue:tasks 0 -1
```

### MongoDB Container Checkpoint
```bash
# Run MongoDB container
docker run -d --name mongo-test \
    -e MONGO_INITDB_ROOT_USERNAME=admin \
    -e MONGO_INITDB_ROOT_PASSWORD=adminpass \
    -v ~/db-lab/mongo-data:/data/db \
    mongo:5.0

# Create test data
docker exec mongo-test mongosh --username admin --password adminpass --authenticationDatabase admin --eval "
use testdb;
db.users.insertMany([
    {name: 'Alice', age: 25, city: 'New York'},
    {name: 'Bob', age: 30, city: 'San Francisco'},
    {name: 'Charlie', age: 35, city: 'Chicago'}
]);
db.products.insertMany([
    {name: 'Laptop', price: 999, category: 'Electronics'},
    {name: 'Book', price: 19, category: 'Education'},
    {name: 'Coffee', price: 5, category: 'Food'}
]);
"

# Checkpoint with database sync
docker exec mongo-test mongosh --username admin --password adminpass --authenticationDatabase admin --eval "db.fsyncLock();"
cedana dump --type containerd --id $(docker ps -q -f name=mongo-test)
docker exec mongo-test mongosh --username admin --password adminpass --authenticationDatabase admin --eval "db.fsyncUnlock();"
```

## Advanced Database Scenarios

### Database Cluster Checkpoint
```bash
# PostgreSQL master-slave setup
docker run -d --name pg-master \
    -e POSTGRES_PASSWORD=masterpass \
    -e POSTGRES_REPLICATION_USER=replica \
    -e POSTGRES_REPLICATION_PASSWORD=replicapass \
    postgres:13

docker run -d --name pg-slave \
    -e PGUSER=replica \
    -e PGPASSWORD=replicapass \
    --link pg-master \
    postgres:13

# Coordinate checkpoint across cluster
# Master checkpoint
cedana dump --type containerd --id $(docker ps -q -f name=pg-master) --name master-checkpoint

# Slave checkpoint (synchronized)
cedana dump --type containerd --id $(docker ps -q -f name=pg-slave) --name slave-checkpoint

# Restore cluster maintaining replication
cedana restore --type containerd --checkpoint-dir ./master-checkpoint
cedana restore --type containerd --checkpoint-dir ./slave-checkpoint --link-to master
```

### Application + Database Coordination
```bash
# Run application with database
docker run -d --name app-db postgres:13 -e POSTGRES_PASSWORD=apppass

docker run -d --name web-app \
    --link app-db:database \
    -e DATABASE_URL=postgresql://postgres:apppass@database:5432/postgres \
    node:14 sh -c "
    npm init -y
    npm install pg express
    node -e \"
    const express = require('express');
    const { Pool } = require('pg');
    const app = express();
    const pool = new Pool({ connectionString: process.env.DATABASE_URL });

    app.get('/', async (req, res) => {
        const result = await pool.query('SELECT NOW()');
        res.json({ time: result.rows[0].now });
    });

    app.listen(3000);
    console.log('App running on port 3000');
    \"
"

# Coordinated checkpoint
cedana dump --type containerd --id $(docker ps -q -f name=app-db) --name db-checkpoint
cedana dump --type containerd --id $(docker ps -q -f name=web-app) --name app-checkpoint

# Restore with dependency order
cedana restore --type containerd --checkpoint-dir ./db-checkpoint
sleep 5  # Wait for database startup
cedana restore --type containerd --checkpoint-dir ./app-checkpoint
```

## Data Validation and Testing

### PostgreSQL Validation
```bash
# Pre-checkpoint data verification
docker exec postgres-test psql -U postgres -d testdb -c "
SELECT
    COUNT(*) as total_rows,
    MAX(id) as max_id,
    MIN(created_at) as earliest,
    MAX(created_at) as latest
FROM test_data;
"

# Post-restore validation
docker exec $(docker ps -q -l) psql -U postgres -d testdb -c "
SELECT
    COUNT(*) as total_rows,
    MAX(id) as max_id,
    MIN(created_at) as earliest,
    MAX(created_at) as latest
FROM test_data;
"

# Data consistency check
docker exec $(docker ps -q -l) psql -U postgres -d testdb -c "
SELECT COUNT(*) FROM test_data WHERE id IS NULL OR name IS NULL;
"
```

### Transaction Integrity Testing
```bash
# Start transaction before checkpoint
docker exec postgres-test psql -U postgres -d testdb -c "
BEGIN;
INSERT INTO test_data (name, value) VALUES ('checkpoint_test', 999);
UPDATE test_data SET value = value + 1 WHERE id < 100;
-- Don't commit yet
"

# Checkpoint with active transaction
cedana dump --type containerd --id $(docker ps -q -f name=postgres-test)

# Verify transaction state after restore
docker exec $(docker ps -q -l) psql -U postgres -d testdb -c "
SELECT COUNT(*) FROM test_data WHERE name = 'checkpoint_test';
SELECT COUNT(*) FROM test_data WHERE value > 100 AND id < 100;
"
```

## Best Practices

### Database Preparation
- Always flush buffers before checkpoint
- Consider checkpoint timing relative to backup schedules
- Handle active connections gracefully
- Test restore procedures regularly

### Data Integrity
- Validate data consistency after restore
- Check for corruption using database tools
- Verify transaction log integrity
- Test application functionality post-restore

### Performance Considerations
- Checkpoint during low-activity periods
- Monitor I/O impact during checkpoint operations
- Plan for storage space requirements
- Consider incremental checkpoint strategies

This tutorial enables reliable database migration and disaster recovery scenarios using container checkpoint technology.