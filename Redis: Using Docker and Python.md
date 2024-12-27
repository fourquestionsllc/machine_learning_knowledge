To set up Redis with Docker and use Python to perform CRUD operations while storing Redis data on the host machine, follow these steps:

### Step 1: Set up Redis with Docker

1. **Pull the Redis image:**

   First, pull the official Redis image from Docker Hub if you haven't already:
   
   ```bash
   docker pull redis
   ```

2. **Create a directory for Redis data on the host machine:**

   Create a directory where you want to store Redis data on the host machine (outside the container). For example:

   ```bash
   mkdir -p /path/to/redis/data
   ```

3. **Run Redis with Docker and mount the directory:**

   Now, run the Redis container, mounting the host directory to the Redis container to persist data:

   ```bash
   docker run --name redis-container -d -p 6379:6379 -v /path/to/redis/data:/data redis
   ```

   Explanation:
   - `--name redis-container`: Names the container `redis-container`.
   - `-d`: Runs the container in the background.
   - `-p 6379:6379`: Maps port 6379 on the host to port 6379 in the container (default Redis port).
   - `-v /path/to/redis/data:/data`: Mounts the host directory `/path/to/redis/data` to the container’s `/data` directory. This is where Redis will store its data.
   
4. **Verify Redis is running:**

   After running the container, you can check if Redis is running by using:

   ```bash
   docker ps
   ```

   You should see the `redis-container` running.

### Step 2: Install Python Redis Client

To interact with Redis from Python, you'll need to install the `redis` package:

```bash
pip install redis
```

### Step 3: Python CRUD Operations

Here’s a Python script that demonstrates CRUD (Create, Read, Update, Delete) operations on Redis using the `redis` package.

```python
import redis

# Connect to the Redis server
client = redis.StrictRedis(host='localhost', port=6379, db=0, decode_responses=True)

# CREATE: Set a key-value pair
client.set('name', 'Alice')

# READ: Get the value of a key
name = client.get('name')
print(f"Name: {name}")

# UPDATE: Update the value of the existing key
client.set('name', 'Bob')

# READ: Get the updated value of the key
name_updated = client.get('name')
print(f"Updated Name: {name_updated}")

# DELETE: Remove a key from the database
client.delete('name')

# TRY READ: Try to get the value of the deleted key
name_deleted = client.get('name')
print(f"Deleted Name: {name_deleted}")  # Should print None
```

### Explanation of CRUD operations:

- **Create**: `client.set('key', 'value')` - Sets a key-value pair in Redis.
- **Read**: `client.get('key')` - Retrieves the value for a key.
- **Update**: `client.set('key', 'new_value')` - Updates the value of an existing key.
- **Delete**: `client.delete('key')` - Deletes a key from Redis.

### Step 4: Docker-Host Data Persistence

Since the Redis data is stored in the mounted directory (`/path/to/redis/data`), it persists even if the Redis container is stopped or removed. You can verify that data is persisted by:

1. Stopping the Redis container:

   ```bash
   docker stop redis-container
   ```

2. Starting it again:

   ```bash
   docker start redis-container
   ```

Redis will continue from where it left off, with data stored in the mounted directory.

---

This setup ensures that the Redis data is stored outside the Docker container, making it persistent across container restarts and allowing easy access to data even if the container is deleted.
