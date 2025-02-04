To migrate the **MLflow Model Registry** from one **Databricks workspace** to another, including model artifacts, follow these steps:

---

### **1. Identify Source and Destination MLflow Instances**
- **Source MLflow instance:** Your current Databricks workspace.
- **Destination MLflow instance:** The target Databricks workspace.

---

### **2. Export Models and Artifacts from Source MLflow Instance**
#### **2.1 Export Model Registry Metadata**
You need to export model names, versions, metadata, tags, and descriptions.

Run the following **Python script** in the source Databricks workspace:

```python
import mlflow
import json
import os

mlflow.set_tracking_uri("databricks")  # Ensure you use Databricks MLflow

def export_model_registry(output_path):
    client = mlflow.tracking.MlflowClient()
    models = client.search_registered_models()

    os.makedirs(output_path, exist_ok=True)
    
    for model in models:
        model_data = model.to_dict()
        model_name = model_data['name']
        
        # Save metadata
        with open(os.path.join(output_path, f"{model_name}.json"), "w") as f:
            json.dump(model_data, f, indent=4)
        
        # Download all versions
        for version in model_data["latest_versions"]:
            model_uri = version["source"]
            local_path = os.path.join(output_path, model_name, f"v{version['version']}")
            os.makedirs(local_path, exist_ok=True)

            # Download the model artifact
            mlflow.artifacts.download_artifacts(model_uri, local_path)

export_model_registry("/dbfs/mnt/mlflow_export")
```
This will export:
1. **Model registry metadata** (e.g., tags, versions).
2. **Model artifacts** (stored in `/dbfs/mnt/mlflow_export`).

---

### **3. Upload Model Artifacts to Destination MLflow Instance**
#### **3.1 Copy Exported Artifacts to a Cloud Storage (S3, ADLS, or GCS)**
Since the **destination MLflow instance** might have different storage settings, you need to move the artifacts.

Example: Upload to Azure Data Lake Storage (ADLS)
```shell
az storage blob upload-batch -d <DESTINATION_CONTAINER> --account-name <STORAGE_ACCOUNT> -s /dbfs/mnt/mlflow_export
```

---

### **4. Import Models into Destination MLflow**
#### **4.1 Register the Models and Upload Artifacts**
In the **destination Databricks workspace**, run:

```python
import mlflow
import json
import os

mlflow.set_tracking_uri("databricks")  # Set tracking URI for destination workspace
client = mlflow.tracking.MlflowClient()

def import_model_registry(input_path):
    for file in os.listdir(input_path):
        if file.endswith(".json"):
            with open(os.path.join(input_path, file), "r") as f:
                model_data = json.load(f)
            
            model_name = model_data["name"]
            client.create_registered_model(model_name)  # Create model registry entry
            
            # Register each version
            for version in model_data["latest_versions"]:
                artifact_path = f"{input_path}/{model_name}/v{version['version']}"
                
                run_id = mlflow.create_experiment(model_name)
                with mlflow.start_run(run_id=run_id):
                    model_info = mlflow.pyfunc.log_model("model", artifact_path=artifact_path)
                    client.create_model_version(model_name, model_info.model_uri, run_id)

import_model_registry("/dbfs/mnt/mlflow_import")
```
This will:
- **Register models** in the new MLflow instance.
- **Upload artifacts** from the cloud storage.

---

### **5. Verify Migration**
Check in the **Databricks UI → Machine Learning → Model Registry** to confirm all models and versions are migrated successfully.

---

### **6. Update Production Endpoints**
If the model is being used in production, update:
- Any **API endpoints** serving the models.
- The **model URI** in Databricks workflows.

---

### **Alternative: MLflow Migration Tool**
Databricks provides an **open-source MLflow Migration Tool**:  
🔗 **https://github.com/mlflow/mlflow-export-import**

```shell
pip install mlflow-export-import

python -m mlflow_export_import.bulk.export_models --models "model1,model2" --output-dir /dbfs/mnt/mlflow_export
python -m mlflow_export_import.bulk.import_models --input-dir /dbfs/mnt/mlflow_import
```
This automates the migration.

---

### **Conclusion**
- **Step 1**: Export models and artifacts.
- **Step 2**: Move artifacts to a shared location (S3/ADLS).
- **Step 3**: Import models into the new MLflow instance.
- **Step 4**: Verify migration and update references.

This ensures a **smooth MLflow Model Registry migration** between Databricks workspaces. 🚀
