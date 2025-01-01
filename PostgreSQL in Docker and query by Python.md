Setting up a PostgreSQL Docker instance and performing CRUD operations via Python involves a few steps. Below, I'll guide you through the process of setting up PostgreSQL in Docker, connecting to it using Python, and performing CRUD operations with SQL queries.

### Step 1: Set Up PostgreSQL Docker Container

To start a PostgreSQL instance in Docker, use the following steps:

1. **Pull the PostgreSQL Docker Image** (if you haven't already):
   ```bash
   docker pull postgres
   ```

2. **Run the PostgreSQL container**:
   You can run a PostgreSQL instance using Docker with the following command:
   ```bash
   docker run --name my_postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
   ```

   - `--name my_postgres`: Name of the container.
   - `-e POSTGRES_PASSWORD=mysecretpassword`: Set the password for the `postgres` user.
   - `-d`: Run the container in detached mode.
   - `postgres`: Use the official PostgreSQL image.

3. **Verify the container is running**:
   ```bash
   docker ps
   ```

4. **Access PostgreSQL**:
   You can access the running PostgreSQL instance using `psql` inside the container or connect remotely via Python.
   To access it via `psql`, run:
   ```bash
   docker exec -it my_postgres psql -U postgres
   ```

   This will open the PostgreSQL prompt.

### Step 2: Install Required Python Libraries

Make sure you have the necessary libraries installed in Python to connect to PostgreSQL:

```bash
pip install psycopg2
```

`psycopg2` is the library that allows Python to interact with PostgreSQL.

### Step 3: Python Code for CRUD Operations

Here is the Python code that connects to your PostgreSQL instance in Docker and performs CRUD (Create, Read, Update, Delete) operations.

```python
import psycopg2
from psycopg2 import sql

# Database connection details
DB_HOST = "localhost"  # If you are running PostgreSQL in Docker
DB_PORT = "5432"       # Default PostgreSQL port
DB_NAME = "postgres"   # Default database name
DB_USER = "postgres"   # Default PostgreSQL user
DB_PASSWORD = "mysecretpassword"  # The password you set in the Docker command

# Establish a connection to the database
conn = psycopg2.connect(
    dbname=DB_NAME,
    user=DB_USER,
    password=DB_PASSWORD,
    host=DB_HOST,
    port=DB_PORT
)

# Create a cursor to execute SQL queries
cursor = conn.cursor()

# 1. Create Table
def create_table():
    query = """
    CREATE TABLE IF NOT EXISTS users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100),
        age INT
    );
    """
    cursor.execute(query)
    conn.commit()
    print("Table created successfully.")

# 2. Insert Data (Create)
def insert_user(name, age):
    query = """
    INSERT INTO users (name, age) VALUES (%s, %s);
    """
    cursor.execute(query, (name, age))
    conn.commit()
    print(f"User {name} inserted successfully.")

# 3. Read Data (Read)
def get_all_users():
    query = "SELECT * FROM users;"
    cursor.execute(query)
    users = cursor.fetchall()
    for user in users:
        print(f"ID: {user[0]}, Name: {user[1]}, Age: {user[2]}")

# 4. Update Data
def update_user(id, new_name, new_age):
    query = """
    UPDATE users
    SET name = %s, age = %s
    WHERE id = %s;
    """
    cursor.execute(query, (new_name, new_age, id))
    conn.commit()
    print(f"User with ID {id} updated successfully.")

# 5. Delete Data
def delete_user(id):
    query = """
    DELETE FROM users WHERE id = %s;
    """
    cursor.execute(query, (id,))
    conn.commit()
    print(f"User with ID {id} deleted successfully.")

# Main program execution
if __name__ == "__main__":
    # Create table (only need to run once)
    create_table()
    
    # Insert users
    insert_user("Alice", 30)
    insert_user("Bob", 25)
    
    # Get and display all users
    print("All users:")
    get_all_users()
    
    # Update a user's data
    update_user(1, "Alice Cooper", 31)
    
    # Get and display all users after update
    print("All users after update:")
    get_all_users()
    
    # Delete a user
    delete_user(2)
    
    # Get and display all users after deletion
    print("All users after deletion:")
    get_all_users()

    # Close the cursor and connection
    cursor.close()
    conn.close()
```

### Explanation of the Code:

1. **Connect to PostgreSQL**: We use `psycopg2.connect()` to connect to the database.
2. **CRUD Operations**:
   - **Create**: `insert_user()` inserts data into the `users` table.
   - **Read**: `get_all_users()` retrieves all users from the `users` table.
   - **Update**: `update_user()` updates a user's name and age.
   - **Delete**: `delete_user()` deletes a user by their `id`.
3. **Commit Changes**: After each SQL operation (`INSERT`, `UPDATE`, `DELETE`), we call `conn.commit()` to save changes to the database.
4. **Close Connections**: Always close the cursor and connection after you're done with them.

### Step 4: Run the Python Code

Simply run the Python script:
```bash
python your_script_name.py
```

This script will interact with the PostgreSQL database running in Docker, create a table, insert records, update records, and delete records.

### Troubleshooting:
- **Connection Issues**: If the connection to Docker is unsuccessful, check that the Docker container is running and accessible at `localhost:5432`.
- **Firewall/Port Mapping**: If you're running Docker on a remote machine, ensure that the PostgreSQL port (`5432`) is properly exposed and mapped.

That's it! You've now set up PostgreSQL in Docker, connected to it using Python, and performed basic CRUD operations.
