# Fix MongoDB Binding for Docker Access

## Problem
MongoDB is only listening on 127.0.0.1, which prevents Docker containers from connecting via host.docker.internal.

## Solution

### Option 1: Configure MongoDB to listen on all interfaces (Recommended)

1. Find your MongoDB configuration file:
   - Windows: `C:\Program Files\MongoDB\Server\<version>\bin\mongod.cfg`
   - Or check MongoDB Compass settings

2. Edit the config file and change:
   ```yaml
   net:
     bindIp: 127.0.0.1
   ```
   
   To:
   ```yaml
   net:
     bindIp: 0.0.0.0
   ```

3. Restart MongoDB service:
   ```cmd
   net stop MongoDB
   net start MongoDB
   ```

### Option 2: Run MongoDB in Docker (Alternative)

Add this stage to your Jenkinsfile before 'Build & Deploy':

```groovy
stage('Start MongoDB') {
    steps {
        bat 'docker run -d --name mongodb -p 27017:27017 mongo:latest || echo "MongoDB already running"'
    }
}
```

Then update backend.env.properties:
```
MONGO_URI=mongodb://host.docker.internal:27017/revhubteam4
```

## Verify the fix

After applying either solution, test with:
```cmd
docker exec backend curl -v telnet://host.docker.internal:27017
```

Should connect successfully.
