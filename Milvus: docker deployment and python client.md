Here's a complete guide on how to deploy Milvus using Docker and interact with it in Python for inserting, querying, and deleting vectors.

---

### **Step 1: Deploy Milvus via Docker**
Run the following command to start Milvus standalone using Docker:
```bash
docker run -d --name milvus-standalone -p 19530:19530 -p 9091:9091 -e ETCD_USE_EMBED=true milvusdb/milvus:v2.3.4
```
- `-p 19530:19530`: Exposes the Milvus service port.
- `-p 9091:9091`: Exposes the Milvus dashboard (optional).
- `milvusdb/milvus:v2.3.4`: Specifies the Milvus version.

Check if Milvus is running:
```bash
docker ps
```

---

### **Step 2: Install the Milvus Python SDK**
Run:
```bash
pip install pymilvus
```

---

### **Step 3: Connect to Milvus and Perform Operations**
Now, create a Python script to interact with Milvus.

#### **Python Code to Insert, Query, and Delete Vectors**
```python
from pymilvus import connections, FieldSchema, CollectionSchema, DataType, Collection

# Connect to Milvus
connections.connect("default", host="localhost", port="19530")

# Define Collection Schema
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="vector", dtype=DataType.FLOAT_VECTOR, dim=128)  # 128-D vector
]
schema = CollectionSchema(fields, description="Example collection")

# Create Collection
collection_name = "example_collection"
collection = Collection(name=collection_name, schema=schema)
collection.load()

# Insert Data
import random
import numpy as np

num_vectors = 10
dim = 128

vectors = np.random.rand(num_vectors, dim).tolist()
data = [
    [i for i in range(num_vectors)],  # IDs
    vectors  # Feature vectors
]
collection.insert(data)

# Create an Index
index_params = {"index_type": "IVF_FLAT", "metric_type": "L2", "params": {"nlist": 1024}}
collection.create_index(field_name="vector", index_params=index_params)

# Search for a Similar Vector
search_vector = np.random.rand(1, dim).tolist()
search_params = {"metric_type": "L2", "params": {"nprobe": 10}}

collection.load()
results = collection.search(search_vector, "vector", search_params, limit=5, output_fields=["id"])
print("Search Results:", results)

# Delete a Vector by ID
delete_id = 3
collection.delete(f"id in [{delete_id}]")
print(f"Deleted vector with ID {delete_id}")

# Drop Collection
collection.drop()
```

---

### **Step 4: Run the Python Script**
Save the script as `milvus_test.py` and execute:
```bash
python milvus_test.py
```

---

### **Explanation of Key Actions**
1. **Deploy Milvus with Docker**
2. **Connect to Milvus** using `pymilvus`
3. **Create a Collection Schema** with a primary key and 128-dimensional float vectors.
4. **Insert Vectors** into the collection.
5. **Index the Data** using `IVF_FLAT` for fast searching.
6. **Search for Similar Vectors** using `L2` distance.
7. **Delete a Vector** by its `id`.
8. **Drop the Collection** when finished.

This setup gives you a fully functional Milvus vector database that you can interact with programmatically from Python. 🚀
