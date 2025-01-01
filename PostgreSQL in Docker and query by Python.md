### Setting Up PostgreSQL Docker Instance with Mapped Directory

#### Step 1: Pull the PostgreSQL Docker Image
Run the following command to pull the PostgreSQL Docker image:
```bash
docker pull postgres
```

#### Step 2: Create a Local Directory for Data Storage
Create a directory on your local system where the PostgreSQL data will be stored:
```bash
mkdir -p ~/postgres-data
```

#### Step 3: Run PostgreSQL Docker Container
Run the PostgreSQL Docker container, mapping the local directory (`~/postgres-data`) to the container's data directory (`/var/lib/postgresql/data`):
```bash
docker run --name postgres-container -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin -e POSTGRES_DB=testdb -v ~/postgres-data:/var/lib/postgresql/data -p 5432:5432 -d postgres
```

This command:
- Names the container `postgres-container`.
- Sets the PostgreSQL username to `admin`, password to `admin`, and database name to `testdb`.
- Maps the local directory `~/postgres-data` to the PostgreSQL data directory inside the container.
- Exposes PostgreSQL on port `5432`.

---

### Querying PostgreSQL with Python

#### Step 1: Install Required Library
Install `psycopg2` for Python to interact with PostgreSQL:
```bash
pip install psycopg2
```

#### Step 2: Python Code for CRUD Operations
Below is the Python script to perform CRUD operations:

```python
import psycopg2

# Database connection settings
DB_HOST = 'localhost'
DB_PORT = '5432'
DB_NAME = 'testdb'
DB_USER = 'admin'
DB_PASSWORD = 'admin'

# Connect to PostgreSQL
def connect_to_db():
    try:
        conn = psycopg2.connect(
            host=DB_HOST,
            port=DB_PORT,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASSWORD
        )
        print("Connection successful")
        return conn
    except Exception as e:
        print(f"Error connecting to database: {e}")
        return None

# Create table
def create_table(conn):
    try:
        with conn.cursor() as cur:
            cur.execute("""
                CREATE TABLE IF NOT EXISTS employees (
                    id SERIAL PRIMARY KEY,
                    name VARCHAR(100),
                    age INT,
                    department VARCHAR(50)
                );
            """)
            conn.commit()
            print("Table created successfully")
    except Exception as e:
        print(f"Error creating table: {e}")

# Insert data
def insert_data(conn, name, age, department):
    try:
        with conn.cursor() as cur:
            cur.execute("""
                INSERT INTO employees (name, age, department)
                VALUES (%s, %s, %s)
                RETURNING id;
            """, (name, age, department))
            new_id = cur.fetchone()[0]
            conn.commit()
            print(f"Inserted data with ID: {new_id}")
    except Exception as e:
        print(f"Error inserting data: {e}")

# Read data
def read_data(conn):
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT * FROM employees;")
            rows = cur.fetchall()
            for row in rows:
                print(row)
    except Exception as e:
        print(f"Error reading data: {e}")

# Update data
def update_data(conn, employee_id, new_department):
    try:
        with conn.cursor() as cur:
            cur.execute("""
                UPDATE employees
                SET department = %s
                WHERE id = %s;
            """, (new_department, employee_id))
            conn.commit()
            print(f"Updated data for ID: {employee_id}")
    except Exception as e:
        print(f"Error updating data: {e}")

# Delete data
def delete_data(conn, employee_id):
    try:
        with conn.cursor() as cur:
            cur.execute("""
                DELETE FROM employees
                WHERE id = %s;
            """, (employee_id,))
            conn.commit()
            print(f"Deleted data for ID: {employee_id}")
    except Exception as e:
        print(f"Error deleting data: {e}")

# Main function
if __name__ == "__main__":
    conn = connect_to_db()
    if conn:
        create_table(conn)
        insert_data(conn, "Alice", 30, "HR")
        insert_data(conn, "Bob", 25, "Engineering")
        print("Initial data:")
        read_data(conn)
        update_data(conn, 1, "Finance")
        print("After update:")
        read_data(conn)
        delete_data(conn, 2)
        print("After delete:")
        read_data(conn)
        conn.close()
```

---

### Explanation of the Code
1. **Connect to Database**: Establishes a connection to the PostgreSQL database using `psycopg2`.
2. **CRUD Operations**:
   - **Create Table**: Creates a table named `employees` if it doesn't already exist.
   - **Insert Data**: Inserts employee records into the table.
   - **Read Data**: Fetches all records from the `employees` table.
   - **Update Data**: Updates a specific record's department field.
   - **Delete Data**: Deletes a specific record based on its ID.
3. **Execution Order**: Demonstrates the CRUD operations sequentially.
