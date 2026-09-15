**Delta Lake** is a **storage layer built on top of data lakes** that makes files such as Parquet behave more like a reliable database table.

It is especially important in **Databricks**.

### Simple picture

Without Delta Lake:

```text
Data Lake
   |
   +-- sales.parquet
   +-- sales_2.parquet
   +-- sales_3.parquet
```

With Delta Lake:

```text
                 Delta Table
                     |
        +------------+------------+
        |                         |
   Parquet data               _delta_log
        |                         |
 actual data files        transaction history
```

The **Parquet files store the actual data**, while the **`_delta_log` records transactions and table metadata**.

### Why do we need Delta Lake?

A traditional data lake can have problems with:

* Updating existing records
* Deleting records
* Concurrent writes
* Schema changes
* Data consistency
* Recovering from failed jobs
* Tracking table history

Delta Lake adds database-like capabilities.

### 1. ACID transactions

Suppose a Spark job is writing 10 million records and fails halfway through.

With a simple Parquet data lake, you can end up with partially written data.

Delta Lake uses transactions so readers see a **consistent table version**.

```text
Before transaction
       ↓
Version 10

Write data
       ↓
Transaction succeeds
       ↓
Version 11
```

If the transaction fails, the incomplete transaction isn't simply exposed as the new table state.

---

### 2. UPDATE / DELETE / MERGE

This is one of the biggest reasons Databricks users use Delta.

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/data/customers")

delta_table.update(
    condition="customer_id = 100",
    set={"status": "'inactive'"}
)
```

You can also do:

```sql
DELETE FROM customers
WHERE customer_id = 100;
```

And:

```sql
UPDATE customers
SET status = 'inactive'
WHERE customer_id = 100;
```

Even more importantly, you can use **MERGE** for upserts:

```sql
MERGE INTO customers AS target
USING new_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
  UPDATE SET *

WHEN NOT MATCHED THEN
  INSERT *;
```

This is extremely common in ETL pipelines.

---

### 3. Time travel

Delta Lake keeps table versions.

For example:

```text
customers
   |
   +-- Version 0
   +-- Version 1
   +-- Version 2
   +-- Version 3  ← current
```

You can query an older version:

```python
df = (
    spark.read
    .format("delta")
    .option("versionAsOf", 2)
    .load("/data/customers")
)
```

Or with SQL:

```sql
SELECT *
FROM customers VERSION AS OF 2;
```

This is called **time travel**.

It's useful when you need to investigate:

> "What did this table look like yesterday?"

or:

> "Why did this customer's value change?"

---

### 4. Schema enforcement

Suppose your table expects:

```text
customer_id
name
amount
```

but a new pipeline suddenly sends:

```text
customer_id
name
amount
random_column
```

Delta can enforce schema rules rather than silently accepting bad structure.

You can also explicitly enable schema evolution when you actually want the schema to change.

---

### 5. Delta + Spark DataFrame

This connects directly to your previous question.

A typical Databricks pipeline looks like:

```text
Raw files
   ↓
Spark DataFrame
   ↓
Transform
   ↓
Delta Lake
   ↓
Delta Table
   ↓
ML / BI / GenAI
```

For example:

```python
df = spark.read.json("/raw/orders")

clean_df = (
    df.filter("amount > 0")
      .dropDuplicates(["order_id"])
)

clean_df.write \
    .format("delta") \
    .mode("append") \
    .save("/data/orders")
```

Then later:

```python
orders = spark.read \
    .format("delta") \
    .load("/data/orders")
```

You can also register it as a table:

```sql
CREATE TABLE orders
USING DELTA
LOCATION '/data/orders';
```

Then:

```sql
SELECT *
FROM orders
WHERE amount > 1000;
```

### Delta Lake vs Parquet

Think of it this way:

|                    | Parquet | Delta Lake |
| ------------------ | ------- | ---------- |
| Columnar storage   | ✅       | ✅          |
| Open format        | ✅       | ✅          |
| ACID transactions  | ❌       | ✅          |
| UPDATE             | Limited | ✅          |
| DELETE             | Limited | ✅          |
| MERGE/upsert       | ❌       | ✅          |
| Time travel        | ❌       | ✅          |
| Schema enforcement | Limited | ✅          |
| Transaction log    | ❌       | ✅          |
| Works with Spark   | ✅       | ✅          |

**Key idea:** Delta Lake does not replace Parquet. **Delta tables typically store data as Parquet and add a transaction log (`_delta_log`) on top.**

So if you're learning the Databricks stack, a very useful progression is:

**Spark DataFrame → Parquet → Delta Lake → Delta Table → Databricks SQL → ML/AI pipelines.**
