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
        model_name = model.name
        model_data = {
            "name": model.name,
            "description": model.description,
            "tags": model.tags,
            "latest_versions": []
        }

        # Save metadata
        model_dir = os.path.join(output_path, model_name)
        os.makedirs(model_dir, exist_ok=True)
        
        for version in model.latest_versions:
            version_info = {
                "version": version.version,
                "status": version.status,
                "run_id": version.run_id,
                "source": version.source,
                "creation_timestamp": version.creation_timestamp,
            }
            model_data["latest_versions"].append(version_info)

            # Download the model artifact
            artifact_path = os.path.join(model_dir, f"v{version.version}")
            os.makedirs(artifact_path, exist_ok=True)
            mlflow.artifacts.download_artifacts(version.source, artifact_path)

        # Save model metadata as JSON
        with open(os.path.join(model_dir, "metadata.json"), "w") as f:
            json.dump(model_data, f, indent=4)

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
    for model_name in os.listdir(input_path):
        model_dir = os.path.join(input_path, model_name)
        metadata_file = os.path.join(model_dir, "metadata.json")

        if not os.path.exists(metadata_file):
            continue

        with open(metadata_file, "r") as f:
            model_data = json.load(f)
        
        # Create the registered model
        try:
            client.create_registered_model(model_name)
        except Exception as e:
            print(f"Model {model_name} might already exist: {str(e)}")

        # Register each version
        for version in model_data["latest_versions"]:
            artifact_path = os.path.join(model_dir, f"v{version['version']}")
            
            # Start a new MLflow run
            with mlflow.start_run():
                model_info = mlflow.pyfunc.log_model("model", artifact_path=artifact_path)
                client.create_model_version(model_name, model_info.model_uri, version["run_id"])

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
