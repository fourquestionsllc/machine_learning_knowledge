Great! Let's get you up and running with **dbt on Windows** and walk through a simple **"Hello World"** example using dbt's built-in **SQLite (DuckDB)** or your preferred local setup.

---

## 🛠️ Step 1: Install dbt on Windows

### ✅ Recommended (Easiest): Install via `pip` with a DuckDB adapter (lightweight and local)

> DuckDB is great for learning and doesn't require a database server.

### 1.1. Install Python (if you don't have it)

- Download Python from: https://www.python.org/downloads/windows/
- Make sure to check **“Add Python to PATH”** during install.

### 1.2. Install dbt and DuckDB adapter

Open Command Prompt or PowerShell:

```bash
pip install dbt-duckdb
```

> You can also install other adapters, like `dbt-postgres`, `dbt-bigquery`, or `dbt-snowflake`, if needed.

---

## 🚀 Step 2: Initialize a dbt Project

Create a folder and navigate into it:

```bash
mkdir my_dbt_project
cd my_dbt_project
```

Initialize the project:

```bash
dbt init my_dbt_project
```

It will ask a few setup questions. Choose:
- **DuckDB** as the adapter
- Use the default settings (or name your DB file)

Navigate to the new project folder:

```bash
cd my_dbt_project
```

---

## 👋 Step 3: Hello World Model

Create a simple model in `models/hello_world.sql`:

**File: `models/hello_world.sql`**
```sql
select 'Hello, dbt on Windows!' as greeting
```

---

## 🏃 Step 4: Run dbt

### 4.1. Compile and run the model:

```bash
dbt run
```

You’ll see:
```
01:00:00  Running with dbt=1.7.6
01:00:01  Found 1 model, 0 tests, 0 snapshots, 0 analyses, 259 macros, 0 operations, 0 seed files, 0 sources, 0 exposures, 0 metrics
01:00:02  
01:00:02  Finished running 1 model in 0.23s.
```

### 4.2. Check the result

You can query the result using DuckDB (optional):

```bash
duckdb my_dbt_project.duckdb
```

Then inside the DuckDB prompt:

```sql
select * from my_dbt_project.hello_world;
```

You should see:

```
greeting
--------------------------
Hello, dbt on Windows!
```

---

## 🧪 Optional: Run a Test

Add a quick test in `models/schema.yml`:

```yaml
version: 2

models:
  - name: hello_world
    columns:
      - name: greeting
        tests:
          - not_null
```

Run:

```bash
dbt test
```

---

## 📄 Optional: Generate Docs

```bash
dbt docs generate
dbt docs serve
```

A browser window will open showing your project’s docs and DAG.

---

## ✅ Summary

| Task                   | Command                      |
|------------------------|------------------------------|
| Install dbt            | `pip install dbt-duckdb`     |
| Create project         | `dbt init my_dbt_project`     |
| Run model              | `dbt run`                    |
| Test model             | `dbt test`                   |
| Serve docs             | `dbt docs serve`             |

---
