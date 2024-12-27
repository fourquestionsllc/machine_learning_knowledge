Here’s how to set up a Redis database using Docker and perform CRUD operations using Python:

---

### **Step 1: Set Up Redis with Docker**
1. **Install Docker**  
   Make sure Docker is installed on your system. [Install Docker](https://www.docker.com/get-started) if not already.

2. **Run Redis Container**  
   Use the following command to pull and run a Redis container:
   ```bash
   docker run --name redis-server -d -p 6379:6379 redis
   ```
   This command:
   - Names the container `redis-server`.
   - Maps the container's Redis port (6379) to the host's port 6379.
   - Runs the latest Redis image in detached mode.

3. **Verify Redis is Running**  
   Use this command to check if the container is running:
   ```bash
   docker ps
   ```
   To test connectivity:
   ```bash
   docker exec -it redis-server redis-cli ping
   ```
   It should return `PONG`.

---

### **Step 2: Install Redis Python Library**
Install the Python Redis library (`redis-py`) using pip:
```bash
pip install redis
```

---

### **Step 3: Perform CRUD Operations in Python**
Here's a Python script to connect to the Redis server and perform basic CRUD operations:

```python
import redis

# Connect to Redis server
redis_client = redis.Redis(host='localhost', port=6379, decode_responses=True)

# Test connection
try:
    redis_client.ping()
    print("Connected to Redis!")
except redis.ConnectionError as e:
    print(f"Redis connection error: {e}")

# CRUD Operations

# Create/Set
redis_client.set("name", "John Doe")
print("Key 'name' set to 'John Doe'.")

# Read/Get
name = redis_client.get("name")
print(f"Retrieved value for 'name': {name}")

# Update
redis_client.set("name", "Jane Doe")
updated_name = redis_client.get("name")
print(f"Updated value for 'name': {updated_name}")

# Delete
redis_client.delete("name")
deleted_name = redis_client.get("name")
print(f"Deleted value for 'name': {deleted_name}")
```

---

### **Additional Redis Operations**
1. **Lists**:
   ```python
   redis_client.lpush("numbers", 1, 2, 3)  # Add to list
   numbers = redis_client.lrange("numbers", 0, -1)  # Get all elements
   print(f"List 'numbers': {numbers}")
   ```

2. **Hashes**:
   ```python
   redis_client.hset("user:1", "name", "Alice")
   redis_client.hset("user:1", "age", 30)
   user = redis_client.hgetall("user:1")
   print(f"Hash 'user:1': {user}")
   ```

3. **Expiration (TTL)**:
   ```python
   redis_client.setex("temp_key", 10, "Temporary Value")  # Key expires in 10 seconds
   print("Temporary key set. It will expire in 10 seconds.")
   ```

---

### **Step 4: Stop and Remove Redis Docker Container**
To stop and remove the Redis container:
```bash
docker stop redis-server
docker rm redis-server
```

This setup allows you to experiment with Redis as an in-memory data store using Docker and Python! 🚀
