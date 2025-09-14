# Multi-Service Live Migration

## What You'll Learn
Migrate complete application stacks (web app + database + cache) with coordinated checkpointing.

## Lab Setup (15 minutes)
```bash
# Create application stack
mkdir app-stack && cd app-stack

# Database service
docker run -d --name stack-db \
    -e POSTGRES_PASSWORD=stackpass \
    -e POSTGRES_DB=appdb \
    postgres:13

# Redis cache
docker run -d --name stack-cache \
    redis:6.2

# Web application that uses both
cat > app.py << EOF
from flask import Flask, jsonify
import psycopg2
import redis
import json
import time

app = Flask(__name__)

# Database connection
db = psycopg2.connect(
    host="stack-db",
    database="appdb",
    user="postgres",
    password="stackpass"
)

# Redis connection
cache = redis.Redis(host="stack-cache", port=6379)

@app.route('/')
def home():
    # Get data from database
    cur = db.cursor()
    cur.execute("SELECT COUNT(*) FROM information_schema.tables")
    table_count = cur.fetchone()[0]

    # Get/set cache data
    visits = cache.incr('visit_count')

    return jsonify({
        'message': 'App Stack Running',
        'database_tables': table_count,
        'cache_visits': visits,
        'timestamp': time.time()
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Create app container
cat > Dockerfile << EOF
FROM python:3.9
RUN pip install flask psycopg2-binary redis
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
EOF

docker build -t app-stack .

# Start app (linked to services)
docker run -d -p 5000:5000 --name stack-app \
    --link stack-db --link stack-cache \
    app-stack
```

## Generate Application State
```bash
# Generate some activity across all services
for i in {1..10}; do
    curl http://localhost:5000/
    sleep 1
done

# Add database data
docker exec stack-db psql -U postgres -d appdb -c "
CREATE TABLE users (id SERIAL, name VARCHAR(50));
INSERT INTO users (name) VALUES ('Alice'), ('Bob'), ('Charlie');
"

# Add cache data
docker exec stack-cache redis-cli SET config:theme "dark"
docker exec stack-cache redis-cli SET config:language "en"
```

## Coordinated Stack Migration
```bash
# Phase 1: Checkpoint all services in dependency order
echo "Checkpointing database..."
cedana dump --type containerd --id $(docker ps -q -f name=stack-db) --name db-checkpoint

echo "Checkpointing cache..."
cedana dump --type containerd --id $(docker ps -q -f name=stack-cache) --name cache-checkpoint

echo "Checkpointing application..."
cedana dump --type containerd --id $(docker ps -q -f name=stack-app) --name app-checkpoint

# Phase 2: Stop all services
docker stop stack-app stack-cache stack-db
docker rm stack-app stack-cache stack-db

# Phase 3: Restore in dependency order
echo "Restoring database..."
cedana restore --type containerd --checkpoint-dir ./db-checkpoint

sleep 5  # Wait for database startup

echo "Restoring cache..."
cedana restore --type containerd --checkpoint-dir ./cache-checkpoint

sleep 3  # Wait for cache startup

echo "Restoring application..."
cedana restore --type containerd --checkpoint-dir ./app-checkpoint
```

## Validate Stack Migration
```bash
# Wait for all services to be ready
sleep 10

# Test application functionality
echo "Testing migrated stack..."
RESPONSE=$(curl -s http://localhost:5000/)
echo $RESPONSE | jq .

# Verify database data
docker exec $(docker ps -q -f ancestor=postgres:13) \
    psql -U postgres -d appdb -c "SELECT COUNT(*) FROM users;"

# Verify cache data
docker exec $(docker ps -q -f ancestor=redis:6.2) \
    redis-cli GET config:theme

# Test continued functionality
curl http://localhost:5000/
curl http://localhost:5000/
```

## What Happens
- Database preserves all tables and data
- Cache maintains all keys and visit counters
- Application resumes with all connections working
- Complete stack operates as if never interrupted

## Advanced: Cross-Host Migration
```bash
# On source host: Create stack checkpoint
./checkpoint-stack.sh

# Transfer checkpoints to target host
rsync -av ./db-checkpoint ./cache-checkpoint ./app-checkpoint target-host:/tmp/

# On target host: Restore complete stack
./restore-stack.sh /tmp/
```

## Test Success
✅ Database data preserved (user tables intact)
✅ Cache state maintained (visit counters continue)
✅ Application responses resume immediately
✅ All inter-service connections work
✅ No data loss across any component

This enables migrating complete application architectures with zero data loss!