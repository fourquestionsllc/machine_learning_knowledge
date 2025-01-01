Here's a step-by-step guide to creating and using a Python-based User-Defined Function (UDF) in PostgreSQL. We'll also set up PostgreSQL using Docker, map a local directory for persistent storage, and install the necessary extensions.

---

### **1. Set Up PostgreSQL with Docker**
Run the following commands to set up PostgreSQL with Docker and map a local directory for data persistence.

#### **Create a Local Directory for PostgreSQL Data**
```bash
mkdir -p ~/postgres_data
```

#### **Run PostgreSQL in Docker**
```bash
docker run --name postgres-python-udf \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=testdb \
  -v ~/postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  -d postgres
```

This will:
- Create a PostgreSQL container named `postgres-python-udf`.
- Map your local `~/postgres_data` directory to the PostgreSQL data directory in the container.

---

### **2. Connect to PostgreSQL**

Use a PostgreSQL client like `psql` to connect:
```bash
psql -h localhost -U admin -d testdb
```

Enter the password (`password`), and you'll be connected to the `testdb` database.

---

### **3. Install the `plpython3u` Extension**
The `plpython3u` extension enables creating UDFs in Python.

Run this command in your PostgreSQL shell:
```sql
CREATE EXTENSION plpython3u;
```

If the extension isn't available, ensure the PostgreSQL Docker image includes Python support or switch to an image like `timescale/timescaledb-ha:pg15`.

---

### **4. Create a Python UDF**
Let's create a UDF that calculates the factorial of a number using Python.

#### **SQL to Define the Function**
```sql
CREATE OR REPLACE FUNCTION calculate_factorial(n INTEGER)
RETURNS BIGINT
LANGUAGE plpython3u
AS $$
import math
if n < 0:
    raise ValueError("Factorial is not defined for negative numbers.")
return math.factorial(n)
$$;
```

This function:
- Takes an integer `n` as input.
- Returns the factorial of `n`.
- Uses Python's `math.factorial`.

---

### **5. Use the UDF in SQL**
You can now call this UDF from SQL queries.

#### **Example Query**
```sql
SELECT calculate_factorial(5);
```

**Output:**
```
 calculate_factorial 
---------------------
                120
(1 row)
```

---

### **6. Docker Notes**

To restart the container, run:
```bash
docker start postgres-python-udf
```

To stop the container, run:
```bash
docker stop postgres-python-udf
```

Your data is persistent because of the mapped `~/postgres_data` directory.

---

### **7. Debugging Tips**
- If the `plpython3u` extension isn’t available, use a PostgreSQL image with Python UDF support.
- Ensure Python 3 is properly configured in the PostgreSQL instance.

This setup allows you to leverage Python for complex computations within PostgreSQL, combining the power of SQL and Python efficiently.
