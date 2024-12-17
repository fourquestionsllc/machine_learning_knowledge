### **SCD Type 1 and Type 2:**

In data warehousing, **Slowly Changing Dimensions (SCD)** refer to the way changes in dimension attributes (data that evolves over time) are handled. There are several strategies for managing these changes, with **Type 1** and **Type 2** being the most common.

---

### **SCD Type 1 (Overwrite)**

**SCD Type 1** is the simplest approach. When a dimension attribute changes, the old value is overwritten by the new value, and **no historical data** is preserved. The existing record is updated directly in the dimension table.

#### **How It Works:**
- When a change occurs (e.g., a customer changes their address or name), the existing dimension row is updated.
- No history of previous values is stored. Only the most current data is kept.

#### **Use Case:**
- This method is appropriate for data where historical changes are not important or when only the current state of the attribute is needed.
- Examples of SCD Type 1 attributes include a **customer's email address** or **contact number**, where updates to these fields don’t require historical tracking.

#### **Example:**

Consider a **Customer** dimension table where a customer's **email address** changes:

| **customer_id** | **first_name** | **last_name** | **email**                |
|-----------------|----------------|---------------|--------------------------|
| 1               | John           | Doe           | john.doe@example.com      |

Now, the customer changes their email to **john.doe@newdomain.com**:

| **customer_id** | **first_name** | **last_name** | **email**                |
|-----------------|----------------|---------------|--------------------------|
| 1               | John           | Doe           | john.doe@newdomain.com    |

- The **old email** is **overwritten** with the new one.

---

### **SCD Type 2 (Add New Row)**

**SCD Type 2** is used when you need to track **historical changes**. When an attribute in a dimension changes, a new row is inserted into the dimension table to reflect the new value, and the old row is **preserved** with its historical data.

- Each row contains a **validity period** (usually in the form of **start date** and **end date**) to indicate when the record was valid.
- A common approach is to add a **current flag** (e.g., `is_current`) to indicate which row is the active, current record.
- This method allows you to track multiple versions of the same record over time.

#### **How It Works:**
- When an attribute changes, instead of updating the existing record, a new record is created, and the old record is flagged as outdated (e.g., by setting an **end date** or **current flag**).
- **Historical tracking** is achieved by maintaining a history of changes for each dimension member.

#### **Use Case:**
- SCD Type 2 is ideal when you need to track the **history of changes** for attributes over time. This is especially useful when analyzing trends or events.
- Examples include tracking **customer addresses** or **employee job positions**, where historical context is essential.

#### **Example:**

Let's say the **address** of a **customer** changes:

1. **Initial record**:
   | **customer_id** | **first_name** | **last_name** | **email**                | **address**              | **start_date** | **end_date** | **is_current** |
   |-----------------|----------------|---------------|--------------------------|--------------------------|----------------|--------------|----------------|
   | 1               | John           | Doe           | john.doe@example.com      | 123 Main St, City A       | 2020-01-01     | NULL         | 1              |

2. **After the address change**:
   - A new record is inserted, and the old record is updated with an end date, indicating the validity period for the old address.

   | **customer_id** | **first_name** | **last_name** | **email**                | **address**              | **start_date** | **end_date**  | **is_current** |
   |-----------------|----------------|---------------|--------------------------|--------------------------|----------------|---------------|----------------|
   | 1               | John           | Doe           | john.doe@example.com      | 123 Main St, City A       | 2020-01-01     | 2023-01-01    | 0              |
   | 1               | John           | Doe           | john.doe@example.com      | 456 Oak St, City B        | 2023-01-02     | NULL          | 1              |

   - The **old address** (`123 Main St, City A`) is now marked as outdated by setting the `end_date` to `2023-01-01` and the `is_current` flag to `0`.
   - A new record is inserted with the updated address (`456 Oak St, City B`), with the `start_date` as the change date and the `is_current` flag set to `1`.

---

### **Key Differences Between SCD Type 1 and Type 2:**

| **Feature**              | **SCD Type 1 (Overwrite)**        | **SCD Type 2 (Add New Row)**                  |
|--------------------------|-----------------------------------|-----------------------------------------------|
| **Historical Data**       | No history of changes is stored.  | History is preserved, and multiple versions of a record are stored. |
| **Storage**               | More efficient; requires less storage. | Requires more storage due to new rows being inserted for each change. |
| **Use Case**              | When only the current value is needed (e.g., email, phone number). | When it is important to track the history of changes (e.g., address, job position). |
| **Data Model**            | Data is updated in place.         | New rows are inserted with valid dates, and current records are flagged. |
| **Complexity**            | Simple to implement and maintain. | More complex; requires tracking dates or flags to indicate the current record. |
| **Query Complexity**      | Simple queries (only current values). | More complex queries to handle historical data and select the current version. |

---

### **Example Use Cases for SCD Type 1 and Type 2**

- **SCD Type 1**: 
  - A customer's **email address** changes, but we don’t need to track the old email. We only need the most up-to-date version.
  - A product's **description** is updated, but we don't need to track the old descriptions.
  
- **SCD Type 2**: 
  - A customer's **address** changes, and we need to track both the old and new address for historical reporting.
  - An **employee's position** changes, and we need to keep records of both their old and current positions for trend analysis.

---

### **Conclusion**

- **SCD Type 1** is simple and works when you don’t need to track historical changes — it just updates the existing record with the new value.
- **SCD Type 2** is more complex but necessary when historical tracking is important — it preserves old records while adding new ones with valid dates to maintain a complete history.

Choosing between Type 1 and Type 2 depends on your business requirements for tracking and reporting historical data.
