# Web Application Container Migration

## What You'll Learn
Migrate running web applications with active user sessions and in-memory state.

## Lab Setup (10 minutes)
```bash
# Create simple web server with state
mkdir webapp-demo && cd webapp-demo

cat > server.js << EOF
const http = require('http');

let counter = 0;
let sessions = {};

const server = http.createServer((req, res) => {
    counter++;

    if (req.url === '/') {
        res.writeHead(200, {'Content-Type': 'text/html'});
        res.end(\`
        <h1>Web App Demo</h1>
        <p>Request count: \${counter}</p>
        <p>Active sessions: \${Object.keys(sessions).length}</p>
        <a href="/session">Create Session</a>
        \`);
    } else if (req.url === '/session') {
        const sessionId = Date.now().toString();
        sessions[sessionId] = { created: new Date(), visits: 1 };
        res.writeHead(200, {'Content-Type': 'text/json'});
        res.end(JSON.stringify({ sessionId, sessions }));
    }
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
    console.log(\`Started at \${new Date()}\`);
});
EOF

# Create Dockerfile
cat > Dockerfile << EOF
FROM node:14-alpine
COPY server.js .
EXPOSE 3000
CMD ["node", "server.js"]
EOF

# Build image
docker build -t webapp-demo .
```

## Run and Generate State
```bash
# Start web application
docker run -d -p 3000:3000 --name webapp webapp-demo

# Generate some application state
curl http://localhost:3000/
curl http://localhost:3000/session
curl http://localhost:3000/session
curl http://localhost:3000/

echo "Check app state at http://localhost:3000"
```

## Checkpoint Web App
```bash
# Get container ID
CONTAINER_ID=$(docker ps -q -f name=webapp)

# Checkpoint running web application
cedana dump --type containerd --namespace moby --id $CONTAINER_ID

# Stop original container
docker stop webapp
docker rm webapp
```

## Restore Web App
```bash
# Restore web application
cedana restore --type containerd --checkpoint-dir ./webapp-checkpoint

# Test that state is preserved
sleep 3
curl http://localhost:3000/

# Request counter and sessions should continue from checkpoint!
```

## What Happens
- In-memory request counter preserved
- Active user sessions maintained
- TCP connections re-established
- Application continues seamlessly

## Test Success
✅ Request counter continues from checkpoint value
✅ Sessions preserved across migration
✅ Web server responds immediately after restore
✅ No data loss or connection errors

Perfect for zero-downtime web application migration!